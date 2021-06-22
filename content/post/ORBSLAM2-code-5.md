---
title: "ORBSLAM2 程式碼解析 - KeyFrame"
date: 2021-06-22T14:42:40+08:00
draft: true
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
