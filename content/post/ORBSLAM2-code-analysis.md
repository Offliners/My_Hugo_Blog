---
title: "ORBSLAM2 程式碼解析"
date: 2021-04-06T00:46:46+08:00
draft: true
toc: false

categories:
  - ORBSLAM
  - code analysis
tags:
  - ORBSLAM
  - code analysis
---

* ORB-SLAM: a Versatile and Accurate Monocular SLAM System
(https://arxiv.org/abs/1610.06475)
* `ORBSLAM2` code : https://github.com/raulmur/ORB_SLAM2
* `ORBSLAM2 for Windows` code: https://github.com/phdsky/ORBSLAM24Windows 

## System Overview
![System Overview](/ORBSLAM2-img/system-overview.JPG)
* `Tracking.cc`
* `LocalMapping.cc`
* `LoopClosing.cc`
* `Viewer.cc`

## 變數命名規則 
|開頭|變數資料型態|
|-|-|
|n|int|
|b|bool|
|p|pointer|
|s|set|
|v|vector|
|l|list|
|m|class member variable|

## System 輸入
![System Input](/ORBSLAM2-img/system_input.JPG)
* 註 : `mpIniORBextractor`比`mpORBextractorLeft`多提取一倍的特徵點
## Tracking 線程
`coming soon`
## LocalMapping 線程
`coming soon`
## LocalClosing 線程(迴路檢測)
`coming soon`
## LocalClosing 線程(Sim3計算)
`coming soon`
## LocalClosing 線程(correctLoop)
`coming soon`
## Frame
`coming soon`
## Monocular Initializer
`coming soon`
## Tracking
`coming soon`
## ORBmatcher
`coming soon`
## Optimizer
`coming soon`

