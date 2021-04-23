---
title: "ORBSLAM2 Build on Windows 10"
date: 2021-04-01T01:31:05+08:00
draft: false
toc: true
comment: true

categories:
  - 環境建立 
tags:
  - ORBSLAM2
  - CMake
  - Visual Studio 2015
  - OpenCV
---

## 測試環境
| Name | Version |
| -------- | -------- |
| Windows 10| `x64` |
| OpenCV | `3.4.1` |
| Visual Studio| `2015` |
| CMake | `3.20` |

* OpenCV : [Install Link](https://opencv.org/releases/)
* Visual Studio 2015 : [Install Link](https://my.visualstudio.com/Downloads?q=visual%20studio%202015&wt.mc_id=o~msft~vscom~older-downloads) 
* CMake : [Install Link](https://cmake.org/download/)

記得將`YOUR_OWN_PATH\opencv\build\x64\vc14\bin`與`YOUR_OWN_PATH\opencv\build`新增至環境變數中，資料夾路徑視OpenCV的安裝目錄而定

## 安裝ORBSLAM2 for Windows
#### 下載 ORBSLAM2
Download : [Link](https://github.com/Phylliida/orbslam-windows)

#### 安裝DBoW2
使用`Cmake-gui`將Source code路徑選`.\orbslam-windows\Thirdparty\DBoW2`，而binaries選擇`.\orbslam-windows\Thirdparty\DBoW2\build`，若沒有`build`資料夾程式可以自動建立

![DBoW2-cmake](/ORBSLAM2-img/DBoW2-cmake-1.JPG)

接著按`Configure`選擇`Visual Studio 14 2015`，平台選擇`x64`或是你的作業系統而定

![DBoW2-cmake-config](/ORBSLAM2-img/DBoW2-cmake-config.JPG)

按下`finish`開始編譯，Configuring done後選擇`Generate`，等Generating done後按`Open Project`開啟VS

進入Visual Studio 2015後，先將Mode選擇`Release`，接著對DBoW2按右鍵選擇Properties，將`Target Extension`的`.dll`改成`.lib`，還有`Configuration Type`改成`Static library(.lib)`

![DBoW2-Properties-1](/ORBSLAM2-img/DBoW2-Properties-1.JPG)

接著，到C/C++的Code Generation將`Runtime Library`改成`Multi-threaded(/MT)`

![DBoW2-Properties-2](/ORBSLAM2-img/DBoW2-Properties-2.JPG)

都選擇好後，對`ALL_BUILD`點右鍵選擇`Build`，建置成功後如下圖

![DBoW2-build](/ORBSLAM2-img/DBoW2-build.JPG)

## 安裝g2o

與DBoW2相同，使用`Cmake-gui`道g2o中編譯，開啟專案後將g2o專案屬性改成`.lib`以及`Multi-threaded(/MT)`，不同的是要到C/C++的`Preprocessor`中在`Preprocessor Definitions`編輯，在最下面輸入`WINDOWS`

![g2o-preprocessor](/ORBSLAM2-img/g2o-preprocessor.JPG)

選擇好後，建置`ALL_BUILD`，建置成功後如下圖

![g2o-build](/ORBSLAM2-img/g2o-build.JPG)

## 安裝Pangolin

依樣使用`Cmake-gui`到Pangolin中進行編譯，進入後只需將Pangolin專案的C/C++的Code Generation將`Runtime Library`改成`Multi-threaded(/MT)`

改完後即可建置`ALL_BUILD`，建置成功後如下圖

![Pangolin-build](/ORBSLAM2-img/Pangolin-build.JPG)

會有一個fail，會顯示"cannot open input file 'pthread.lib'"，忽略即可

## 建置ORBSLAM2
使用`Cmake-gui`將Source code選擇`/orbslam-windows`，且binaries選擇`/orbslam-windows/build`進行編譯

編譯成功後開起專案，對`ORB_SLAM2`專案按右鍵，將屬性改成`.lib`以及`/MT`，改好後建置`ORB_SLAM2`，不是`ALL_BUILD`，建置成功如下

![ORB_SLAM2_build](/ORBSLAM2-img/ORB_SLAM2_build.JPG)

## 使用ORBSLAM2範例程式
首先將`orbslam-windows\Vocabulary`中的`ORBvoc.txt.tar.gz`解壓縮得到`ORBvoc.txt`

再來到 https://vision.in.tum.de/data/datasets 下載黃色標記的資料集

![rgbd_dataset_freiburg2_desk](/ORBSLAM2-img/rgbd_dataset_freiburg2_desk.JPG)

將其解壓縮後將整個資料夾放到`orbslam-windows\`下

接著，打開`orbslam-windows\build`的`ORB_SLAM2.sln`，對`mono_tum`專案點右鍵進行建置，建置成功後可以在`orbslam-windows\Examples\Monocular\Release`找到`mono_tum.exe`，可透過CMD去執行此程式，記得輸入3個參數

* path_to_vocabulary --- `YOUR_OWN_PATH\Vocabulary\ORBvoc.txt`
* path_to_settings --- `YOUR_OWN_PATH\Examples\Monocular\TUM2.yaml`
* path_to_sequence --- `YOUR_OWN_PATH\rgbd_dataset_freiburg2_desk`

CMD輸入方式如下:
```shell
mono_tum path_to_vocabulary path_to_settings path_to_sequence
```

執行結果

![mono_tum](/ORBSLAM2-img/mono_tum.gif)

