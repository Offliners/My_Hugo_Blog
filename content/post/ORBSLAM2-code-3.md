---
title: "ORBSLAM2 程式碼解析 - MapPoint"
date: 2021-04-28T14:04:05+08:00
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

MapPoint是地圖中的特徵點，自身參數是三為座標與描述子，在此類別中會完成以下工作 : 
* 維護關鍵幀之間的共視關係
* 通過計算描述向量之間的距離，在多個關鍵幀的特徵點中找到最匹配的特徵點
* 在閉環完成修正後，需要根據修正的主幀位姿來修正特徵點
* 對於非關鍵幀，也能產生MapPoint，只是是用來給Tracking功能臨時使用

以下為MapPoint中的主要函數
```
MapPoint
│
├── MapPoint  // 建構函數
├──────── 關鍵幀地圖點建構函數
├──────── 普通幀地圖點建構函數
│
├── AddObservation  // 增加地圖點觀測關係
│
├── EraseObservation  // 刪除地圖點觀測關係
│
├── SetBadFlag  // 刪除地圖點
│
├── Replace  // 替換地圖點
│
├── ComputeDistinctiveDescriptors  // 計算描述子
│
├── UpdateNormalAndDepth  // 更新法向量和深度值
│
├── PredictScale  // 預測尺度
```

## 建構函數 MapPoint
一共有兩個，分別對應關鍵幀與普通幀

#### 關鍵幀的地圖點建構函數
```C++
MapPoint(const cv::Mat &Pos, KeyFrame *pRefKF, Map* pMap)
```

Pos為該點的3D位置，pRefKF是參考關鍵幀，pMap是地圖。關鍵幀的地圖點建構函數主要是突出地圖點與關鍵幀之間的關係，一個地圖點會被多個關鍵幀偵測到，多個關鍵幀之間透過共同偵測到的地圖點來產生關係，叫做共視關係。在ORBSLAM中，是透過MapPoint來維護共視關係。在進行BA優化時，指優化具有共視關係的關鍵幀，其他關鍵幀的位姿則不參與優化

#### 普通幀的地圖點建構函數
```C++
MapPoint(const cv::Mat &Pos, Map* pMap, Frame* pFrame, const int &idxF)
```

Pos為該點的3D位置，pMap是地圖，pFrame是對應的普通幀，idxF是地圖點在該幀特徵點的索引值

## 增加地圖點的觀測關係
```C++
void MapPoint::AddObservation(KeyFrame* pKF, size_t idx)
{
    unique_lock<mutex> lock(mMutexFeatures);
    
    // 判斷是否已經存在觀測關係
    if(mObservations.count(pKF))
        return;  // 已存在則返回
    mObservations[pKF]=idx;  // 不存在則添加

    // 分成一元與二元兩種來添加觀測數目
    if(pKF->mvuRight[idx]>=0)
        nObs+=2;  // 二元觀測數目加2
    else
        nObs++;  // 一元觀測數目加1
}
```

這裡有兩個重要的變數 : 
#### std::map<KeyFrame*,size_t> mObservations
此變數用來存放觀測關係，它儲存觀測到該MapPoint的關鍵幀與MapPoint在該關鍵幀中對應的索引值

#### int nObs
此變數用來記錄被觀測到的次數

## 刪除觀測關係
```C++
void MapPoint::EraseObservation(KeyFrame* pKF)
{
    bool bBad=false;
    {
        unique_lock<mutex> lock(mMutexFeatures);
        
        // 判斷該關鍵幀是否存在觀測關係，即該關鍵幀是否看到這個MapPoint
        if(mObservations.count(pKF))
        {
            int idx = mObservations[pKF];
            
            // 分成一元與二元兩種來減少觀測數目
            if(pKF->mvuRight[idx]>=0)
                nObs-=2;  // 二元觀測數目減2
            else
                nObs--;  // 一元觀測數目減1

            mObservations.erase(pKF);  // 刪除該關鍵幀對應的觀測關係

            // 如果該關鍵幀是參考關鍵幀的話，則重新指定
            if(mpRefKF==pKF)
                mpRefKF=mObservations.begin()->first;

            // 當被觀測次數小於等於2時，該地圖需要被剔除
            if(nObs<=2)
                bBad=true;
        }
    }

    if(bBad)
        SetBadFlag();  // 剔除地圖
}
```

參數pKF為關鍵幀。此函數先判斷該關鍵幀是否在觀測中，如果在則從存放觀測關係的mObservations中移除，接著判斷是否為參考關鍵幀，如果是則將參考關鍵幀換成觀測的第一幀，因為不能沒有參考關鍵幀。刪除關鍵幀後，若該MapPoint被觀測數目小於等於2時，則該MapPoint就沒有存在的必要，需要剔除

## 刪除地圖點
```C++
void MapPoint::SetBadFlag()
{
    map<KeyFrame*,size_t> obs;
    {
        unique_lock<mutex> lock1(mMutexFeatures);
        unique_lock<mutex> lock2(mMutexPos);
        mbBad=true;
        obs = mObservations;
        mObservations.clear();  // 清除該地圖點所有的觀測關係
    }
    for(map<KeyFrame*,size_t>::iterator mit=obs.begin(), mend=obs.end(); mit!=mend; mit++)
    {
        KeyFrame* pKF = mit->first;
        pKF->EraseMapPointMatch(mit->second);  // 刪除關鍵幀中和該MapPoint對應的匹配關係
    }

    mpMap->EraseMapPoint(this);  // 從地圖中刪除MapPoint
}
```

此函數主要用來刪除地圖點，並清除關鍵幀與地圖中所有與該地圖點對應的關聯關係

## 替換地圖點
```C++
void MapPoint::Replace(MapPoint* pMP)
{
    // 如果傳入的MapPoint就是該MapPoint則直接跳出
    if(pMP->mnId==this->mnId)
        return;

    int nvisible, nfound;
    map<KeyFrame*,size_t> obs;
    {
        unique_lock<mutex> lock1(mMutexFeatures);
        unique_lock<mutex> lock2(mMutexPos);
        obs=mObservations;
        mObservations.clear();
        mbBad=true;
        nvisible = mnVisible;
        nfound = mnFound;
        mpReplaced = pMP;
    }

    for(map<KeyFrame*,size_t>::iterator mit=obs.begin(), mend=obs.end(); mit!=mend; mit++)
    {
        KeyFrame* pKF = mit->first;

        // 如果輸入的MapPoint不在關鍵幀的觀測關係中，則添加觀測關係
        if(!pMP->IsInKeyFrame(pKF))
        {
            pKF->ReplaceMapPointMatch(mit->second, pMP);
            pMP->AddObservation(pKF,mit->second);
        }
        // 如果在關鍵幀的觀測關係中，就刪除關鍵幀和舊的MapPoint之間的對應關係
        else
        {
            pKF->EraseMapPointMatch(mit->second);
        }
    }
    pMP->IncreaseFound(nfound);
    pMP->IncreaseVisible(nvisible);
    pMP->ComputeDistinctiveDescriptors();

    mpMap->EraseMapPoint(this);  // 刪除Map中該地圖點
}

```

參數pMP就是用來替換的地圖點。此函數要將當前地圖點(this)替換成pMP，因為在使用閉環時，完成閉環以後需要調整地圖點與關鍵幀來建立新關係。具體流程是循遍所有觀測訊息，判斷輸入的MapPoint是否在該關鍵幀中，如果在就只要移除舊MapPoint的匹配訊息，再增加這個MapPoint找到的數量、可見次數與計算這個該點獨有的描述子即可，最後將舊MapPoint移除

## 計算最匹配的描述子
```C++
void MapPoint::ComputeDistinctiveDescriptors()
{
    vector<cv::Mat> vDescriptors;

    map<KeyFrame*,size_t> observations;

    {
        unique_lock<mutex> lock1(mMutexFeatures);
        
        // 如果地圖標記為不好則直接返回
        if(mbBad)
            return;
        observations=mObservations;
    }

    // 如果觀測為空則返回
    if(observations.empty())
        return;

    // 保留的描述子數量最多與觀測數量相同
    vDescriptors.reserve(observations.size());

    for(map<KeyFrame*,size_t>::iterator mit=observations.begin(), mend=observations.end(); mit!=mend; mit++)
    {
        KeyFrame* pKF = mit->first;

        if(!pKF->isBad())
            vDescriptors.push_back(pKF->mDescriptors.row(mit->second));  // 針對每幀對應的都提取其描述子
    }

    if(vDescriptors.empty())
        return;

    const size_t N = vDescriptors.size();

    float Distances[N][N];
    for(size_t i=0;i<N;i++)
    {
        Distances[i][i]=0;
        for(size_t j=i+1;j<N;j++)
        {
            int distij = ORBmatcher::DescriptorDistance(vDescriptors[i],vDescriptors[j]);
            Distances[i][j]=distij;
            Distances[j][i]=distij;
        }
    }

    // 選擇其他描述子中距離最小的描述子作為地圖點的描述子，基本上類似取平均值
    int BestMedian = INT_MAX;
    int BestIdx = 0;
    for(size_t i=0;i<N;i++)
    {
        vector<int> vDists(Distances[i],Distances[i]+N);
        sort(vDists.begin(),vDists.end());
        int median = vDists[0.5*(N-1)];

        if(median<BestMedian)
        {
            BestMedian = median;
            BestIdx = i;
        }
    }

    {
        unique_lock<mutex> lock(mMutexFeatures);
        mDescriptor = vDescriptors[BestIdx].clone();
    }
}
```

一個MapPoint會被許多相機觀測到，因此在插入關鍵幀後，需要判斷是否更新當前點最適合的描述子。最好的描述子與其他描述子應該會有最小的平均距離，因此先取得當前點所有的描述子，然後計算描述子之間的距離再取平均，最後找離這個均值最近的描述子

## 更新法向量與深度值
```C++
void MapPoint::UpdateNormalAndDepth()
{
    map<KeyFrame*,size_t> observations;
    KeyFrame* pRefKF;
    cv::Mat Pos;
    {
        unique_lock<mutex> lock1(mMutexFeatures);
        unique_lock<mutex> lock2(mMutexPos);
        if(mbBad)
            return;
        observations=mObservations;
        pRefKF=mpRefKF;
        Pos = mWorldPos.clone();
    }

    if(observations.empty())
        return;

    cv::Mat normal = cv::Mat::zeros(3,1,CV_32F);
    int n=0;
    for(map<KeyFrame*,size_t>::iterator mit=observations.begin(), mend=observations.end(); mit!=mend; mit++)
    {
        KeyFrame* pKF = mit->first;
        cv::Mat Owi = pKF->GetCameraCenter();
        cv::Mat normali = mWorldPos - Owi;  // 觀測點座標減去關鍵幀中相機光心的座標就是觀測方向，也就是相機光心指向地圖點的方向
        normal = normal + normali/cv::norm(normali);  // 對其進行歸一化後相加
        n++;
    }

    cv::Mat PC = Pos - pRefKF->GetCameraCenter();
    const float dist = cv::norm(PC);
    const int level = pRefKF->mvKeysUn[observations[pRefKF]].octave;
    const float levelScaleFactor =  pRefKF->mvScaleFactors[level];
    const int nLevels = pRefKF->mnScaleLevels;

    {
        unique_lock<mutex> lock3(mMutexPos);
        mfMaxDistance = dist*levelScaleFactor;
        mfMinDistance = mfMaxDistance/pRefKF->mvScaleFactors[nLevels-1];
        mNormalVector = normal/n;
    }
}
```

由於圖像提取描述子是使用金字塔分層提取，所以計算法向量和深度可以知道該MapPoint對應的關鍵幀可以從金字塔哪一層中提取到。而法向量就是相機光心指向地圖點的方向，計算方法就是將地圖點的三D座標減去相機光心的3D座標即可

而深度範圍為地圖點到參考幀(只有1幀)相機中心的距離，乘上參考幀中描述子獲得的金字塔放大尺度就可得最大距離`mfMaxDistance`。最大距離除以整個金字塔最高層的放大尺度就可以得到最小距離`mfMinDistance`。通常來說，距離較近的地圖點將在金字塔較高的地方提取出;反之，距離較遠的地圖點將在金字塔較低的地方提取出(金字塔曾數越低，分辨率愈高，因此能識別出遠近)。透過地圖點訊息(主要是對應的描述子)，可以獲得該地圖點對應的金字塔層級，進而預測該地圖在什麼範圍內能被觀測到

## 預測尺度
```C++
int MapPoint::PredictScale(const float &currentDist, KeyFrame* pKF)
{
    float ratio;
    {
        unique_lock<mutex> lock(mMutexPos);
        ratio = mfMaxDistance/currentDist;
    }

    int nScale = ceil(log(ratio)/pKF->mfLogScaleFactor);
    if(nScale<0)
        nScale = 0;
    else if(nScale>=pKF->mnScaleLevels)
        nScale = pKF->mnScaleLevels-1;

    return nScale;
}
```

參數currentDist為當前距離，pKF為關鍵幀。此函數用來預測特徵點在金字塔哪一層可以找到

以下為示意圖 : 
![預測尺度](/ORBSLAM2-img/scale.JPG)
