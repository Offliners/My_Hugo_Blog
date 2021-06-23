---
title: "ORBSLAM2 程式碼解析 - KeyFrame"
date: 2021-06-22T14:42:40+08:00
draft: false
toc: true
comment: true
description: ORBSLAM2 Code Analysis - KeyFrame

categories:
  - ORBSLAM2 程式碼解析
tags:
  - ORBSLAM2
  - code analysis
---

## System Overview
![System Overview](/ORBSLAM2-img/system-overview.JPG)

#### Source Code : [Link](https://github.com/raulmur/ORB_SLAM2)

KeyFrame為關鍵幀，主要用來優化圖像，因此內部函式都是圖像的優化服務。要優化圖像需要有節點與邊，節點就是關鍵幀的位姿，因此需要有讀寫關鍵幀位姿的功能。而邊分成兩種，一種是連結MapPoint，所以需要有管理與MapPoint之間關係的函數，另一種是連結其他關鍵幀，由於兩者都需要連結MapPoint，所以當兩者共同觀測到一定數量的MapPoint時則在兩者之間建立邊來連結，這種關係叫做共視，因此需要有管理共視關係的函數。藉由共視關係建立的優化模型叫做`Covisibility Graph`，當需要優化較大範圍的數據時，就需要大量的計算量，因此在ORBSLAM2中的`Essential Graph`就是簡化版，透過生成樹(Spanning Tree)來建構優化模型，並將擁有父子關係的關鍵幀建立邊，所以還需要有管理生成樹的函式

![Graph](/ORBSLAM2-img/graph.JPG)
在[ORB-SLAM: a Versatile and Accurate Monocular SLAM System](https://arxiv.org/abs/1502.00956)這篇論文中有上圖可以比較，圖(b)是每個節點是一個關鍵幀。若兩關鍵幀之間共視的MapPoint數量大於15時則建立邊。圖(c)就是生成樹，幫每個關鍵幀找父節點與子節點，每幀個只與自帶的父子節點相連，與其他幀則不相連。因此圖(d)就可以根據圖(c)來建立簡易的圖模型

以下為KeyFrame類別中的主要函數 : 
```
KeyFrame
│
├── KeyFrame  // 建構函數
│
├── // 位姿相關
├────── SetPose  // 設置位姿
├────── GetPose  // 獲取位姿
├────── GetPoseInverse  // 獲取位姿的逆
├────── GetCamercaCenter  // 獲取傳感器中心的座標
├────── GetStereoCenter  // 獲取雙目的中心座標
├────── GetRotation  // 獲取旋轉矩陣
├────── GetTranslation  // 獲取平移矩陣
│
├── // Covisibility Graph相關
├────── AddConnection  // 增加連結
├────── EraseConnection  // 刪除連結
├────── UpdateConnection  // 更新連結
├────── UpdateBestCovisibles  // 更新最佳的Covisibles
├────── GetConnectedKeyFrames  // 獲取連接的關鍵幀
├────── GetVectorCovisibleKeyFrames  // 獲取Covisible關鍵幀
├────── GetBestCovisibleKeyFrame  // 獲取最佳的Covisibility關鍵幀
├────── GetCovisibleByWeight  // 根據權重獲得Covisibility
├────── GetWeight  // 獲取權重
├────── SetNoErase  // 設置不要移除
├────── SetErase  // 設置要移除
├────── SetBadFlag  // 刪除關鍵幀以及所有關係
├────── isBad  // 判斷是否需要移除
├────── AddLoopEdge  // 增加回環的邊
├────── GetLoopEdges  // 獲取回環的邊
│
├── // Spanning Tree相關
├────── AddChild  // 增加子樹
├────── EraseChild  // 刪除子樹
├────── ChangeParent  // 更改父節點
├────── GetChild  // 獲取子節點
├────── GetParent  // 獲取父節點
├────── hasChild  // 判斷是否有子節點
│
├── // MapPoint相關
├────── AddMapPoint  // 添加MapPoint
├────── EraseMapPointMatch  // 移除MapPoint的匹配關係(使用索引)
├────── EraseMapPointMatch  // 移除MapPoint的匹配關係(使用MapPoint)
├────── ReplaceMapPointMatch  // 更替MapPoint的匹配
├────── TrackedMapPoint  // 跟蹤MapPoint
├────── GetMapPointMatch  // 獲取MapPoint的匹配
├────── GetMapPoints  // 獲取所有MapPoint
├────── GetMapPoint  // 獲取指定MapPoint
│
├── // Other
├────── ComputeBoW  // 利用詞袋計算特徵
├────── GetFeatureInArea  // 獲取特定區域內的特徵點
├────── UnprojectStereo  // 獲取雙目特徵點的3D座標
├────── IsInImage  // 判斷是否在圖像中
├────── ComputeSceneMedianDepth  // 計算場景中的中位深度
├────── weightComp  // 權重的比較
├────── lId  // 比較兩個關鍵幀的幀號
```

## 建構函數
```C++
KeyFrame::KeyFrame(Frame &F, Map *pMap, KeyFrameDatabase *pKFDB):
    mnFrameId(F.mnId),  mTimeStamp(F.mTimeStamp), mnGridCols(FRAME_GRID_COLS), mnGridRows(FRAME_GRID_ROWS),
    mfGridElementWidthInv(F.mfGridElementWidthInv), mfGridElementHeightInv(F.mfGridElementHeightInv),
    mnTrackReferenceForFrame(0), mnFuseTargetForKF(0), mnBALocalForKF(0), mnBAFixedForKF(0),
    mnLoopQuery(0), mnLoopWords(0), mnRelocQuery(0), mnRelocWords(0), mnBAGlobalForKF(0),
    fx(F.fx), fy(F.fy), cx(F.cx), cy(F.cy), invfx(F.invfx), invfy(F.invfy),
    mbf(F.mbf), mb(F.mb), mThDepth(F.mThDepth), N(F.N), mvKeys(F.mvKeys), mvKeysUn(F.mvKeysUn),
    mvuRight(F.mvuRight), mvDepth(F.mvDepth), mDescriptors(F.mDescriptors.clone()),
    mBowVec(F.mBowVec), mFeatVec(F.mFeatVec), mnScaleLevels(F.mnScaleLevels), mfScaleFactor(F.mfScaleFactor),
    mfLogScaleFactor(F.mfLogScaleFactor), mvScaleFactors(F.mvScaleFactors), mvLevelSigma2(F.mvLevelSigma2),
    mvInvLevelSigma2(F.mvInvLevelSigma2), mnMinX(F.mnMinX), mnMinY(F.mnMinY), mnMaxX(F.mnMaxX),
    mnMaxY(F.mnMaxY), mK(F.mK), mvpMapPoints(F.mvpMapPoints), mpKeyFrameDB(pKFDB),
    mpORBvocabulary(F.mpORBvocabulary), mbFirstConnection(true), mpParent(NULL), mbNotErase(false),
    mbToBeErased(false), mbBad(false), mHalfBaseline(F.mb/2), mpMap(pMap)
{
    mnId=nNextId++;

    mGrid.resize(mnGridCols);  // 根據網格的行數來重置網格大小
    for(int i=0; i<mnGridCols;i++)
    {
        mGrid[i].resize(mnGridRows);
        for(int j=0; j<mnGridRows; j++)
            mGrid[i][j] = F.mGrid[i][j];
    }

    SetPose(F.mTcw);  // 將關鍵幀的姿態傳給該關鍵幀
}
```
此建構函數輸入參數為
* Frame &F : 當前幀
* Map *pMap : 地圖
* KeyFrameDatabase *pKFDB : 關鍵幀數據集的指標

## 位姿相關
### SetPose
```C++
void KeyFrame::SetPose(const cv::Mat &Tcw_)
{
    unique_lock<mutex> lock(mMutexPose);
    Tcw_.copyTo(Tcw);
    cv::Mat Rcw = Tcw.rowRange(0,3).colRange(0,3);
    cv::Mat tcw = Tcw.rowRange(0,3).col(3);
    cv::Mat Rwc = Rcw.t();
    Ow = -Rwc*tcw;  // 相機的光心

    Twc = cv::Mat::eye(4,4,Tcw.type());
    Rwc.copyTo(Twc.rowRange(0,3).colRange(0,3));
    Ow.copyTo(Twc.rowRange(0,3).col(3));
    cv::Mat center = (cv::Mat_<float>(4,1) << mHalfBaseline, 0 , 0, 1);
    Cw = Twc*center;
}
```
此函數輸入參數為
* Tcw_ : 當前幀的位姿

### GetPos
```C++
cv::Mat KeyFrame::GetPose()
{
    unique_lock<mutex> lock(mMutexPose);
    return Tcw.clone();
}
```

### GetPoseInverse
```C++
cv::Mat KeyFrame::GetPoseInverse()
{
    unique_lock<mutex> lock(mMutexPose);
    return Twc.clone();
}
```

### GetCamercaCenter
```C++
cv::Mat KeyFrame::GetCameraCenter()
{
    unique_lock<mutex> lock(mMutexPose);
    return Ow.clone();
}
```

### GetStereoCenter
```C++
cv::Mat KeyFrame::GetStereoCenter()
{
    unique_lock<mutex> lock(mMutexPose);
    return Cw.clone();
}
```

### GetRotation
```C++
cv::Mat KeyFrame::GetRotation()
{
    unique_lock<mutex> lock(mMutexPose);
    return Tcw.rowRange(0,3).colRange(0,3).clone();
}
```

### GetTranslation
```C++
cv::Mat KeyFrame::GetTranslation()
{
    unique_lock<mutex> lock(mMutexPose);
    return Tcw.rowRange(0,3).col(3).clone();
}
```

### Covisibility Graph相關
### AddConnection
### EraseConnection
### UpdateConnection
### UpdateBestCovisibles
### GetConnectedKeyFrames
### GetVectorCovisibleKeyFrames
### GetBestCovisibleKeyFrame
### GetCovisibleByWeight
### GetWeight
### SetNoErase
### SetErase
### SetBadFlag
### isBad
### AddLoopEdge
### GetLoopEdges

## Spanning Tree相關
### AddChild
### EraseChild
### ChangeParent
### GetChild
### GetParent
### hasChild

## MapPoint相關
### AddMapPoint
### EraseMapPointMatch
### EraseMapPointMatch
### ReplaceMapPointMatch
### TrackedMapPoint
### GetMapPointMatch
### GetMapPoints
### GetMapPoint

## Other
### ComputeBoW
### GetFeatureInArea
### UnprojectStereo
### IsInImage
### ComputeSceneMedianDepth
### weightComp
### lId