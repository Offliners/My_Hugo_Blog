---
title: "ORBSLAM2 程式碼解析 - Frame"
date: 2021-04-28T20:50:07+08:00
draft: false
toc: true
comment: true

categories:
  - ORBSLAM2 程式碼解析
tags:
  - ORBSLAM2
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

## 提取ORB特徵點
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

## 設置相機位姿參數
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

設置相機位姿參數，並計算相機光心的位置

## 判斷路標點是否在視野中
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

## 查找特定區域內的特徵點
```C++
vector<size_t> Frame::GetFeaturesInArea(const float &x, const float  &y, const float  &r, const int minLevel, const int maxLevel) const
{
    vector<size_t> vIndices;
    vIndices.reserve(N);

    // 計算正方形的四邊在網格中的行列數
    // nMinCellX 是正方形左邊在mGrid中的列數，如果大於mGrid的列數的話，表示正方形內沒有特徵點，所以返回
    const int nMinCellX = max(0,(int)floor((x-mnMinX-r)*mfGridElementWidthInv));
    if(nMinCellX>=FRAME_GRID_COLS)
        return vIndices;

    const int nMaxCellX = min((int)FRAME_GRID_COLS-1,(int)ceil((x-mnMinX+r)*mfGridElementWidthInv));
    if(nMaxCellX<0)
        return vIndices;

    const int nMinCellY = max(0,(int)floor((y-mnMinY-r)*mfGridElementHeightInv));
    if(nMinCellY>=FRAME_GRID_ROWS)
        return vIndices;

    const int nMaxCellY = min((int)FRAME_GRID_ROWS-1,(int)ceil((y-mnMinY+r)*mfGridElementHeightInv));
    if(nMaxCellY<0)
        return vIndices;

    const bool bCheckLevels = (minLevel>0) || (maxLevel>=0);

    for(int ix = nMinCellX; ix<=nMaxCellX; ix++)
    {
        for(int iy = nMinCellY; iy<=nMaxCellY; iy++)
        {
            const vector<size_t> vCell = mGrid[ix][iy];
            if(vCell.empty())
                continue;

            for(size_t j=0, jend=vCell.size(); j<jend; j++)
            {
                const cv::KeyPoint &kpUn = mvKeysUn[vCell[j]];
                if(bCheckLevels)
                {
                    if(kpUn.octave<minLevel)
                        continue;
                    if(maxLevel>=0)
                        if(kpUn.octave>maxLevel)
                            continue;
                }

                const float distx = kpUn.pt.x-x;
                const float disty = kpUn.pt.y-y;

                // 把區域內所有特徵點放入容器中
                if(fabs(distx)<r && fabs(disty)<r)
                    vIndices.push_back(vCell[j]);
            }
        }
    }

    return vIndices;
}
```

此函數用來找以x，y為中心，邊長為2r的正方形且在minLevel~maxLevel的特徵點

## 利用二元恢復深度
```C++
void Frame::ComputeStereoMatches()
{
    mvuRight = vector<float>(N,-1.0f);
    mvDepth = vector<float>(N,-1.0f);

    const int thOrbDist = (ORBmatcher::TH_HIGH+ORBmatcher::TH_LOW)/2;

    const int nRows = mpORBextractorLeft->mvImagePyramid[0].rows;

    // Step 1 : 建立特徵點搜索範圍對應表，一個特徵點在一個帶狀區域內搜索匹配特徵點
    // 匹配搜索時，不只在一條橫線上搜索，而是在一條橫向搜索帶上搜索，簡言之，原本每個特徵點的縱座標為1，這裡把特徵點體積放大，縱座標橫跨好幾行
    // 例如 : 左目圖像的某個特徵點縱座標為20，那麼右目圖像上搜索時會在縱座標18~22這範圍內搜索，搜索帶範圍為正負2，而搜索帶的寬度與特徵點與所在金字塔層數有關
    vector<vector<size_t> > vRowIndices(nRows,vector<size_t>());

    for(int i=0; i<nRows; i++)
        vRowIndices[i].reserve(200);

    const int Nr = mvKeysRight.size();

    // 把所有特徵點對應的y值都設置一個搜索帶，然後把搜索帶內所有y座標都和對應的特徵點座標做關聯
    for(int iR=0; iR<Nr; iR++)
    {
        const cv::KeyPoint &kp = mvKeysRight[iR];
        const float &kpY = kp.pt.y;
        
        // 計算匹配搜索帶的縱向寬度，尺度越大(層數越高，距離越近)，搜索帶範圍越大
        // 如果特徵點在金字塔第一層時，則搜索範圍為正負2
        // 尺度越大旗位置不確定性越高，所以搜索半徑越大
        const float r = 2.0f*mvScaleFactors[mvKeysRight[iR].octave];
        const int maxr = ceil(kpY+r);
        const int minr = floor(kpY-r);

        for(int yi=minr;yi<=maxr;yi++)
            vRowIndices[yi].push_back(iR);
    }

    // Set limits for search
    const float minZ = mb;  // Bug : mb沒有初始化，指派mb值的建構函數在ComputeStereoMatches函數後
    const float minD = 0;  // 最小視差，設置為0即可
    const float maxD = mbf/minZ;  // 最大視差，對應最小深度，mbf/minZ = mbf/mb = mbf/(mbf/fx) = fx

    vector<pair<int, int> > vDistIdx;
    vDistIdx.reserve(N);

    // Step 2 : 對左目相機每個特徵點，通過描述子在右目帶狀搜索區域中找到匹配點，再通過SAD做亞像素匹配
    // 注意 : 這裡是使用校正前的mvKeys，而不是校正後的mvKeysUn
    // 注意 : KeyFrame::UnprojectStereo與Frame::UnprojectStereo是不同的
    for(int iL=0; iL<N; iL++)
    {
        const cv::KeyPoint &kpL = mvKeys[iL];
        const int &levelL = kpL.octave;
        const float &vL = kpL.pt.y;
        const float &uL = kpL.pt.x;

        const vector<size_t> &vCandidates = vRowIndices[vL];  // 可能的匹配點

        if(vCandidates.empty())
            continue;

        const float minU = uL-maxD;  // 最小匹配範圍
        const float maxU = uL-minD;  // 最大匹配範圍

        if(maxU<0)
            continue;

        int bestDist = ORBmatcher::TH_HIGH;
        size_t bestIdxR = 0;

        const cv::Mat &dL = mDescriptors.row(iL);  // 每個特徵點描述子使用一行，建立一個指標指向iL特徵點對應的描述子

        // Step 2-1 : 尋遍右目所有可能的匹配點，找到最佳匹配點(描述子距離最小)
        for(size_t iC=0; iC<vCandidates.size(); iC++)
        {
            const size_t iR = vCandidates[iC];
            const cv::KeyPoint &kpR = mvKeysRight[iR];

            // 只對鄰近尺度的特徵點進行匹配
            if(kpR.octave<levelL-1 || kpR.octave>levelL+1)
                continue;

            const float &uR = kpR.pt.x;

            // 找出bestIdxR就是最匹配的特徵點，bestDict是該特徵點對應的描述向量距離
            if(uR>=minU && uR<=maxU)
            {
                const cv::Mat &dR = mDescriptorsRight.row(iR);
                const int dist = ORBmatcher::DescriptorDistance(dL,dR);

                if(dist<bestDist)
                {
                    bestDist = dist;
                    bestIdxR = iR;
                }
            }
        }

        // Step 2-2 : 通過SAD匹配來提高像素匹配修正量bestincR
        if(bestDist<thOrbDist)
        {
            // kpL.pt.x對應金字塔最底層的座標，將最佳匹配特徵點的尺度轉換到尺度對應層
            const float uR0 = mvKeysRight[bestIdxR].pt.x;
            const float scaleFactor = mvInvScaleFactors[kpL.octave];
            const float scaleduL = round(kpL.pt.x*scaleFactor);
            const float scaledvL = round(kpL.pt.y*scaleFactor);
            const float scaleduR0 = round(uR0*scaleFactor);

            // 滑動窗口大小為11*11
            // 注意 : 該窗口取自Resize後的圖像
            const int w = 5;
            cv::Mat IL = mpORBextractorLeft->mvImagePyramid[kpL.octave].rowRange(scaledvL-w,scaledvL+w+1).colRange(scaleduL-w,scaleduL+w+1);
            IL.convertTo(IL,CV_32F);
            IL = IL - IL.at<float>(w,w) *cv::Mat::ones(IL.rows,IL.cols,CV_32F);

            int bestDist = INT_MAX;
            int bestincR = 0;
            const int L = 5;
            vector<float> vDists;
            vDists.resize(2*L+1);

            // 滑動窗口的滑動範圍為(-L，L)，提前判斷滑動窗口過程是否發生越界
            const float iniu = scaleduR0+L-w;
            const float endu = scaleduR0+L+w+1;
            if(iniu<0 || endu >= mpORBextractorRight->mvImagePyramid[kpL.octave].cols)
                continue;

            for(int incR=-L; incR<=+L; incR++)
            {
                // 橫向滑動窗口
                cv::Mat IR = mpORBextractorRight->mvImagePyramid[kpL.octave].rowRange(scaledvL-w,scaledvL+w+1).colRange(scaleduR0+incR-w,scaleduR0+incR+w+1);
                IR.convertTo(IR,CV_32F);
                
                // 窗口中的每個元素減去中心的元素，並歸一化，來減少高照強度的影響
                IR = IR - IR.at<float>(w,w) *cv::Mat::ones(IR.rows,IR.cols,CV_32F);

                float dist = cv::norm(IL,IR,cv::NORM_L1);
                if(dist<bestDist)
                {
                    bestDist =  dist;  // SAD匹配目前最小匹配偏差
                    bestincR = incR;  // SAD匹配目前最佳的修正量
                }

                vDists[L+incR] = dist;
            }

            // 整個滑動過程中，SAD值不是以拋物線形式呈現，則表示SAD匹配失敗，同時放棄求該特徵點深度
            if(bestincR==-L || bestincR==L)
                continue;

            // Step 2-3 : 做拋物線擬合來找谷底得到亞像素匹配deltaR
            // bestincR+deltaR就是拋物線谷底的位置
            const float dist1 = vDists[L+bestincR-1];
            const float dist2 = vDists[L+bestincR];
            const float dist3 = vDists[L+bestincR+1];

            const float deltaR = (dist1-dist3)/(2.0f*(dist1+dist3-2.0f*dist2));

            // 拋物線擬合得到的修正量不能超過一個像素，超過則放氣球該特徵點的深度
            if(deltaR<-1 || deltaR>1)
                continue;

            // 通過描述子匹配得到位置為scaleduR0
            // 通過SAD匹配找到修正量bestincR
            // 通過拋物線擬合找到亞像素修正量deltaR
            float bestuR = mvScaleFactors[kpL.octave]*((float)scaleduR0+(float)bestincR+deltaR);

            float disparity = (uL-bestuR);

            // 判斷視差是否在範圍內
            if(disparity>=minD && disparity<maxD)
            {
                if(disparity<=0)
                {
                    disparity=0.01;
                    bestuR = uL-0.01;
                }
                
                // 深度 depth = baseline * fx / disparity
                mvDepth[iL]=mbf/disparity;  // 深度 
                mvuRight[iL] = bestuR;  // 匹配在右圖的橫坐標
                vDistIdx.push_back(pair<int,int>(bestDist,iL));  // 該特徵點SAD匹配最小偏差
            }
        }
    }

    // Step 3 : 去除SAD匹配偏差較大的特徵點
    // 之前SAD匹配指判斷滑動窗口是否有局部最小值，這裡會根據對比去除SAD偏差較大特徵點的深度
    sort(vDistIdx.begin(),vDistIdx.end());  // 根據所有匹配對的SAD偏差進行排序，由小到大
    const float median = vDistIdx[vDistIdx.size()/2].first;
    const float thDist = 1.5f*1.4f*median;  // 計算自適應距離，大於此距離的匹配則會被去除

    for(int i=vDistIdx.size()-1;i>=0;i--)
    {
        if(vDistIdx[i].first<thDist)
            break;
        else
        {
            mvuRight[vDistIdx[i].second]=-1;
            mvDepth[vDistIdx[i].second]=-1;
        }
    }
}
```

此函數是為左圖的每一個特徵點找到在右圖的匹配點，根據基線(有冗餘範圍)上描述子的距離找到匹配，再進行SAD精確定位，最後對所有SAD的值進行排序，去除SAD較大的匹配對，然後利用拋物線擬何來得到亞像素精度的匹配，匹配成功後會更新mvuRight與mvDepth

## RGBD獲取深度訊息
```C++
void Frame::ComputeStereoFromRGBD(const cv::Mat &imDepth)
{
    mvuRight = vector<float>(N,-1);
    mvDepth = vector<float>(N,-1);

    for(int i=0; i<N; i++)
    {
        const cv::KeyPoint &kp = mvKeys[i];
        const cv::KeyPoint &kpU = mvKeysUn[i];

        const float &v = kp.pt.y;
        const float &u = kp.pt.x;

        const float d = imDepth.at<float>(v,u);

        if(d>0)
        {
            mvDepth[i] = d;
            mvuRight[i] = kpU.pt.x-mbf/d;
        }
    }
}
```

根據像素座標獲取深度訊息並保存起來，這裡還計算假想右圖對應特徵點的橫坐標

## 特徵點座標反投影到3D地圖點
```C++
cv::Mat Frame::UnprojectStereo(const int &i)
{
    const float z = mvDepth[i];
    if(z>0)
    {
        const float u = mvKeysUn[i].pt.x;
        const float v = mvKeysUn[i].pt.y;
        const float x = (u-cx)*z*invfx;
        const float y = (v-cy)*z*invfy;
        cv::Mat x3Dc = (cv::Mat_<float>(3,1) << x, y, z);
        return mRwc*x3Dc+mOw;
    }
    else
        return cv::Mat();
}
```

此函數作用是將特徵點座標反投影到3D地圖點(世界座標)，在已知深度的情況下，可以確定二維像素對應的尺度，最後獲得3D地圖中的點座標
