---
title: "ORBSLAM2 程式碼解析 - Frame"
date: 2021-04-28T20:50:07+08:00
draft: false
toc: true
comment: true

categories:
  - ORBSLAM2 程式碼解析
tags:
  - ORBSLAM
  - code analysis
---

## System Overview
![System Overview](/ORBSLAM2-img/system-overview.JPG)

#### Source Code : [Link](https://github.com/raulmur/ORB_SLAM2)

Frame是指幀，也就是對應的圖像，可以是一元、二元或RGBD，所以此類別所包含SLAM中以幀為單位的操作，有以下方面 : 
* 讀寫該幀對應的相機位姿
* 處理幀和特徵點之間的關係，包括判斷特徵點是否在視野內、獲取該幀依定區域內的特徵點、校正特徵點等
* 恢復深度，如果是RGBD就直接讀取深度值，如果是一元或二元則進行深度恢復

以下為Frame類別中的主要函數 : 
```
Frame
│
├── Frame  // 建構函數，有以下五種
├────── 預設建構函數
├────── 複製建構函數
├────── 二元幀建構函數
├────── RGBD幀建構函數
├────── 一元幀建構函數
│
├── ExtractORB  // 提取ORB特徵點
│
├── ComputeBoW  // 計算詞袋數據
│
├── SetPose  // 設置相機位姿參數
│
├── isInFrustum  // 判斷路標點是否在視野中
│
├── GetFeaturesInArea  // 查找特定區域內的特徵點
│
├── ComputeStereoMatches  // 利用二元恢復深度
│
├── ComputeStereoFromRGBD  // RGBD獲取深度訊息
│
├── UnprojectStereo  // 特徵點座標反投影到3D地圖點
```

## 建構函數
建構函數除了預設與複製之外，其他主要對應三種相機模型，主要功能也類似，就是提取特徵點，然後把特徵點劃分到網格中，使特徵點在圖像中分布較均勻

另外，還有深度問題，RGBD相機本身有深度值直接讀取即可，二元相機需使用SAD來恢復深度，而一元相機無法獲取深度，所以深度值直接為-1，這些都在建構函數中完成

#### 二元相機
```C++
Frame::Frame(const cv::Mat &imLeft, const cv::Mat &imRight, const double &timeStamp, ORBextractor* extractorLeft, ORBextractor* extractorRight, ORBVocabulary* voc, cv::Mat &K, cv::Mat &distCoef, const float &bf, const float &thDepth)
    :mpORBvocabulary(voc),mpORBextractorLeft(extractorLeft),mpORBextractorRight(extractorRight), mTimeStamp(timeStamp), mK(K.clone()),mDistCoef(distCoef.clone()), mbf(bf), mThDepth(thDepth),
     mpReferenceKF(static_cast<KeyFrame*>(NULL))
{
    mnId=nNextId++;

    mnScaleLevels = mpORBextractorLeft->GetLevels();
    mfScaleFactor = mpORBextractorLeft->GetScaleFactor();
    mfLogScaleFactor = log(mfScaleFactor);
    mvScaleFactors = mpORBextractorLeft->GetScaleFactors();
    mvInvScaleFactors = mpORBextractorLeft->GetInverseScaleFactors();
    mvLevelSigma2 = mpORBextractorLeft->GetScaleSigmaSquares();
    mvInvLevelSigma2 = mpORBextractorLeft->GetInverseScaleSigmaSquares();

    // 同時對左右目提取特徵
    thread threadLeft(&Frame::ExtractORB,this,0,imLeft);
    thread threadRight(&Frame::ExtractORB,this,1,imRight);
    threadLeft.join();
    threadRight.join();

    N = mvKeys.size();

    // mvKeys存放提取的特徵點，如果沒有的話則退出
    if(mvKeys.empty())
        return;

    UndistortKeyPoints();  // 對特徵點進行畸變校正

    ComputeStereoMatches();  // 計算左右目之間的匹配，匹配成功的特徵點會計算其深度，深度會存放在mvuRight和mvDepth中

    mvpMapPoints = vector<MapPoint*>(N,static_cast<MapPoint*>(NULL));  // 對應的MapPoints 
    mvbOutlier = vector<bool>(N,false);


    // 第一次進入或者標定文件發生變化時會使用該函數，重新計算相機相關參數
    if(mbInitialComputations)
    {
        ComputeImageBounds(imLeft);

        mfGridElementWidthInv=static_cast<float>(FRAME_GRID_COLS)/(mnMaxX-mnMinX);
        mfGridElementHeightInv=static_cast<float>(FRAME_GRID_ROWS)/(mnMaxY-mnMinY);

        fx = K.at<float>(0,0);
        fy = K.at<float>(1,1);
        cx = K.at<float>(0,2);
        cy = K.at<float>(1,2);
        invfx = 1.0f/fx;
        invfy = 1.0f/fy;

        mbInitialComputations=false;
    }

    mb = mbf/fx;

    AssignFeaturesToGrid();  // 把特徵點劃分到網格中，可以設置網格內的特徵點上限，使特徵點分布更均勻
}
```

此建構函數輸入參數為
* const cv::Mat &imLeft  // 左目圖像
* const cv::Mat &imRight  // 右目圖像
* const double &timeStamp  // 時間戳
* ORBextractor* extractorLeft  // 左目提取特徵
* ORBextractor* extractorRight  // 右目提取特徵
* ORBVocabulary* voc  // 詞袋數據
* cv::Mat &K  // 相機內部參數
* cv::Mat &distCoef  // 圖像校正參數
* const float &bf  // bf = 二元基線 * fx
* const float &thDepth  // 深度的閥值，特徵點深度大於或小於此值時，會被分為close與far兩類

#### RGBD相機
```C++
Frame::Frame(const cv::Mat &imGray, const cv::Mat &imDepth, const double &timeStamp, ORBextractor* extractor,ORBVocabulary* voc, cv::Mat &K, cv::Mat &distCoef, const float &bf, const float &thDepth)
    :mpORBvocabulary(voc),mpORBextractorLeft(extractor),mpORBextractorRight(static_cast<ORBextractor*>(NULL)),
     mTimeStamp(timeStamp), mK(K.clone()),mDistCoef(distCoef.clone()), mbf(bf), mThDepth(thDepth)
{
    mnId=nNextId++;

    mnScaleLevels = mpORBextractorLeft->GetLevels();
    mfScaleFactor = mpORBextractorLeft->GetScaleFactor();   
    mfLogScaleFactor = log(mfScaleFactor);
    mvScaleFactors = mpORBextractorLeft->GetScaleFactors();
    mvInvScaleFactors = mpORBextractorLeft->GetInverseScaleFactors();
    mvLevelSigma2 = mpORBextractorLeft->GetScaleSigmaSquares();
    mvInvLevelSigma2 = mpORBextractorLeft->GetInverseScaleSigmaSquares();

    ExtractORB(0,imGray);  // 提取特徵點

    N = mvKeys.size();

    if(mvKeys.empty())
        return;

    UndistortKeyPoints();  // 對特徵點進行畸變校正

    ComputeStereoFromRGBD(imDepth);  // 根據像素座標獲得深度訊息，如果存在則保存起來，函數內還計算假想右圖所對應特徵點的橫坐標

    mvpMapPoints = vector<MapPoint*>(N,static_cast<MapPoint*>(NULL));
    mvbOutlier = vector<bool>(N,false);

    if(mbInitialComputations)
    {
        ComputeImageBounds(imGray);

        mfGridElementWidthInv=static_cast<float>(FRAME_GRID_COLS)/static_cast<float>(mnMaxX-mnMinX);
        mfGridElementHeightInv=static_cast<float>(FRAME_GRID_ROWS)/static_cast<float>(mnMaxY-mnMinY);

        fx = K.at<float>(0,0);
        fy = K.at<float>(1,1);
        cx = K.at<float>(0,2);
        cy = K.at<float>(1,2);
        invfx = 1.0f/fx;
        invfy = 1.0f/fy;

        mbInitialComputations=false;
    }

    mb = mbf/fx;

    AssignFeaturesToGrid();  // 把特徵點劃分到網格中
}
```

此建構函數輸入參數為
* const cv::Mat &imGray  // 灰階圖
* const cv::Mat &imDepth  // 深度值
* const double &timeStamp  // 時間戳
* ORBextractor* extractor  // ORB特徵提取
* ORBVocabulary* voc  // 詞袋數據
* cv::Mat &K  // 相機內部參數
* cv::Mat &distCoef  // 圖像校正參數
* const float &bf  // bf = 二元基線 * fx 
* const float &thDepth  // 深度的閥值，特徵點深度大於或小於此值時，會被分為close與far兩類

#### 一元相機
```C++
Frame::Frame(const cv::Mat &imGray, const double &timeStamp, ORBextractor* extractor,ORBVocabulary* voc, cv::Mat &K, cv::Mat &distCoef, const float &bf, const float &thDepth)
    :mpORBvocabulary(voc),mpORBextractorLeft(extractor),mpORBextractorRight(static_cast<ORBextractor*>(NULL)),
     mTimeStamp(timeStamp), mK(K.clone()),mDistCoef(distCoef.clone()), mbf(bf), mThDepth(thDepth)
{
    mnId=nNextId++;

    mnScaleLevels = mpORBextractorLeft->GetLevels();
    mfScaleFactor = mpORBextractorLeft->GetScaleFactor();
    mfLogScaleFactor = log(mfScaleFactor);
    mvScaleFactors = mpORBextractorLeft->GetScaleFactors();
    mvInvScaleFactors = mpORBextractorLeft->GetInverseScaleFactors();
    mvLevelSigma2 = mpORBextractorLeft->GetScaleSigmaSquares();
    mvInvLevelSigma2 = mpORBextractorLeft->GetInverseScaleSigmaSquares();

    ExtractORB(0,imGray);

    N = mvKeys.size();

    if(mvKeys.empty())
        return;

    UndistortKeyPoints();

    // 沒有右目，所以值為-1
    mvuRight = vector<float>(N,-1);
    mvDepth = vector<float>(N,-1);

    mvpMapPoints = vector<MapPoint*>(N,static_cast<MapPoint*>(NULL));
    mvbOutlier = vector<bool>(N,false);

    if(mbInitialComputations)
    {
        ComputeImageBounds(imGray);

        mfGridElementWidthInv=static_cast<float>(FRAME_GRID_COLS)/static_cast<float>(mnMaxX-mnMinX);
        mfGridElementHeightInv=static_cast<float>(FRAME_GRID_ROWS)/static_cast<float>(mnMaxY-mnMinY);

        fx = K.at<float>(0,0);
        fy = K.at<float>(1,1);
        cx = K.at<float>(0,2);
        cy = K.at<float>(1,2);
        invfx = 1.0f/fx;
        invfy = 1.0f/fy;

        mbInitialComputations=false;
    }

    mb = mbf/fx;

    AssignFeaturesToGrid();
}
```

此建構函數輸入參數為
* const cv::Mat &imGray  // 灰階圖
* const double &timeStamp  // 時間戳
* ORBextractor* extractor  // ORB特徵提取
* ORBVocabulary* voc  // 詞袋數據
* cv::Mat &K  // 相機內部參數
* cv::Mat &distCoef  // 圖像校正參數
* const float &bf  // bf = 二元基線 * fx 
* const float &thDepth  // 深度的閥值，特徵點深度大於或小於此值時，會被分為close與far兩類

## 提取特徵點
```C++
void Frame::ExtractORB(int flag, const cv::Mat &im)
{
    if(flag==0)
        (*mpORBextractorLeft)(im,cv::Mat(),mvKeys,mDescriptors);
    else
        (*mpORBextractorRight)(im,cv::Mat(),mvKeysRight,mDescriptorsRight);
}
```

此函數將OpenCV自身的ORB提取功能多封裝一層，增加一個flag來決定提取左目或右目，進而使用不同的特徵提取器

## 計算詞袋數據
```C++
void Frame::ComputeBoW()
{
    if(mBowVec.empty())
    {
        vector<cv::Mat> vCurrentDesc = Converter::toDescriptorVector(mDescriptors);
        mpORBvocabulary->transform(vCurrentDesc,mBowVec,mFeatVec,4);
    }
}
```

如果沒有輸入已有的詞袋數據，則用當前的描述子重新計算來生成詞袋數據

## 設置相機外部參數
```C++
void Frame::SetPose(cv::Mat Tcw)
{
    mTcw = Tcw.clone();
    UpdatePoseMatrices();
}

void Frame::UpdatePoseMatrices()
{ 
    mRcw = mTcw.rowRange(0,3).colRange(0,3);
    mRwc = mRcw.t();
    mtcw = mTcw.rowRange(0,3).col(3);
    mOw = -mRcw.t()*mtcw;
}
```

設置相機外部參數，並計算相機光心的位置

## 判斷該MapPoint是否在當前幀的視野中
```C++
bool Frame::isInFrustum(MapPoint *pMP, float viewingCosLimit)
{
    pMP->mbTrackInView = false;

    cv::Mat P = pMP->GetWorldPos(); 

    // 3D點P在相機坐標系下的位置
    const cv::Mat Pc = mRcw*P+mtcw;
    const float &PcX = Pc.at<float>(0);
    const float &PcY= Pc.at<float>(1);
    const float &PcZ = Pc.at<float>(2);

    if(PcZ<0.0f)
        return false;

    // 將MapPoint投影到當前的幀
    const float invz = 1.0f/PcZ;
    const float u=fx*PcX*invz+cx;
    const float v=fy*PcY*invz+cy;

    // 判斷投影後的坐標是否在圖像中
    if(u<mnMinX || u>mnMaxX)
        return false;
    if(v<mnMinY || v>mnMaxY)
        return false;

    // 計算MapPoint到相機中心的距離，並判斷是否在尺度變化的距離內
    // 每一個地圖點都是從若干尺度的金字塔中提取出來的，具有一定的有效深度
    const float maxDistance = pMP->GetMaxDistanceInvariance();
    const float minDistance = pMP->GetMinDistanceInvariance();
    const cv::Mat PO = P-mOw;
    const float dist = cv::norm(PO);

    if(dist<minDistance || dist>maxDistance)
        return false;

    // 計算當前視角和平均視角夾角的餘弦值，若小於cos(60°)，即夾角大於60度則返回
    // 每一個地圖都有平均視角，能夠從觀測到地圖點的位姿計算出來
    cv::Mat Pn = pMP->GetNormal();
    const float viewCos = PO.dot(Pn)/dist;
    if(viewCos<viewingCosLimit)
        return false;

    // 根據深度來預測尺度(對應特徵點在哪一層)
    const int nPredictedLevel = pMP->PredictScale(dist,this);

    // 如果在視野範圍內，由於tracking會用到，因此此處要給值
    pMP->mbTrackInView = true;  // 標記位置為true，在函數開頭預設為false
    pMP->mTrackProjX = u;
    pMP->mTrackProjXR = u - mbf*invz;  // 該3D點投影到二元右側相機上的橫坐標
    pMP->mTrackProjY = v;
    pMP->mnTrackScaleLevel= nPredictedLevel;
    pMP->mTrackViewCos = viewCos;

    return true;
}

```

先計算MapPoint在相機坐標系下的位置，用該點與相機光心的連線可知道它在相機的哪個視角範圍內(即該連線與相機正前方的夾角)，如果這角度大於設定值則認為該點不在視野內，反之則在，在的話就計算該MapPoint在該幀圖像上的座標，以便跟踪時使用

## 獲得特定區域內的座標點
```C++

```
