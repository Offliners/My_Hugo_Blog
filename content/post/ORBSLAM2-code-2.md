---
title: "ORBSLAM2 程式碼解析 - 系統流程"
date: 2021-04-27T19:39:14+08:00
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

系統流程的入口在`System.cc`，裡面有4個主要的函數 : 
* System
* TrackStereo
* TrackRGBD
* TrackMonocular

其中`System`是SLAM系統的建構函數，包括所有功能模塊與所有線程的初始化。而`TrackStereo`、`TrackRGBD`、`TrackMonocular`分別為二元、深度與一元相機的數據入口，使用ORBSLAM時會先透過System來生成SLAM系統的使用對象，然後根據傳感器的類型來選擇相對應的入口來傳入數據

## 建構函數 System
```
System // 建構函數
│
├── mpVocabulary = new ORBVocabulary();  // 加載ORB詞袋模型
│
│── mpKeyFrameDatabase = new KeyFrameDatabase(*mpVocabulary);
│   // 創建關鍵幀數據集，數據集中主要存放詞袋模型中的值，這些值用於閉環檢測與重定位
│
│── mpMap = new Map();  // 創建地圖
│
│── mpFrameDrawer = new FrameDrawer(mpMap);  // 畫關鍵幀的Drawer
│
│── mpMapDrawer = new MapDrawer(mpMap, strSettingsFile);  // 畫地圖的Drawer
│
│── mpTracker = new Tracking(this, mpVocabulary, mpFrameDrawer, mpMapDrawer, mpMap, mpKeyFrameDatabase, strSettingsFile, mSensor);
│   // 建立跟踪線程
│
│── mpLocalMapper = new LocalMapping(mpMap, mSensor==MONOCULAR);
│   // 建立局部地圖線程
│
│── mpLoopCloser = new LoopClosing(mpMap, mpKeyFrameDatabase, mpVocabulary, mSensor!=MONOCULAR);
│   // 建立局部閉環線程
```

建構函數中和流程相關的輸入變數有4個 : 
* strVocFile : ORB詞袋數據，詞袋是在做閉環檢測與重定位時做檢索用，從歷史關鍵幀中檢索出和目前幀最相似的一幀，以進行位姿匹配
* strSettingsFile : 配置文件的路徑，配置文件中存放相機參數與用戶顯示介面的設置
* sensor : 傳感器類型，這是個枚舉變量，總共有STEREO、RGBD與MONOCULAR三種，分別代表二元、深度與一元
* bUseViewer : true表示顯示介面，false表示不顯示

建構函數初始化以下內容 : 
#### 初始化詞袋模型
```C++
mpVocabulary = new ORBVocabulary();  // 初始化詞袋對象
bool bVocLoad = mpVocabulary->loadFromTextFile(strVocFile);  // 從詞袋數據文件中讀取數據
```
#### 初始化關鍵幀數據集
```C++
mpKeyFrameDatabase = new KeyFrameDatabase(*mpVocabulary);  // 使用詞袋數據集建立數據集對象
```

在`KeyFrameDatabase.cc`中可以發現此模組除了加載數據，還實現閉環檢測與重定位檢測
```C++
KeyFrameDatabase::DetectLoopCandidates(KeyFrame* pKF, float minScore)  // 檢測閉環向量
KeyFrameDatabase::DetectRelocalizationCandidates(Frame *F)  // 檢測重定位向量
```

詳細內容在閉環模組時會說明

#### 初始化地圖
```C++
mpMap = new Map();
```

地圖中主要存放關鍵幀與特徵點

#### 初始化用戶顯示介面，並開啟對應的線程
```C++
mpFrameDrawer = new FrameDrawer(mpMap);  // 畫關鍵幀
mpMapDrawer = new MapDrawer(mpMap, strSettingsFile);  // 畫地圖

/*
    ...
*/

if(bUseViewer)  // 根據設定判斷是否需顯示介面
{
    mpViewer = new Viewer(this,mpFrameDrawer,mpMapDrawer,mpTracker,strSettingsFile);  // 顯示介面
    mptViewer = new thread(&Viewer::Run, mpViewer);
    mpTracker->SetViewer(mpViewer);  // 開啟對應的線程
}
```

#### 初始化局部地圖，並開啟對應的線程
```C++
mpLocalMapper = new LocalMapping(mpMap, mSensor==MONOCULAR);  // 初始化局部地圖
mptLocalMapping = new thread(&ORB_SLAM2::LocalMapping::Run,mpLocalMapper);  // 開啟對應的線程
```

#### 初始化局部地圖，並開啟對應的線程
```C++
mpLoopCloser = new LoopClosing(mpMap, mpKeyFrameDatabase, mpVocabulary, mSensor!=MONOCULAR);  // 初始化局部地圖
mptLoopClosing = new thread(&ORB_SLAM2::LoopClosing::Run, mpLoopCloser);  // 開啟對應的線程
```

#### 透過指標建立各線程間的連結
```C++
mpTracker->SetLocalMapper(mpLocalMapper);
mpTracker->SetLoopClosing(mpLoopCloser);

mpLocalMapper->SetTracker(mpTracker);
mpLocalMapper->SetLoopCloser(mpLoopCloser);

mpLoopCloser->SetTracker(mpTracker);
mpLoopCloser->SetLocalMapper(mpLocalMapper);
```

## 二元相機入口 TrackStereo
```
TrackStereo
│
├── if(mbActivateLocalizationMode)  // 檢測是否開啟純定位模式
├──────────────── mpLocalMapper->RequestStop();  // 停止局部地圖建構
├──────────────── mpTracker->InformOnlyTracking(true);  // 設置成純定位模式
│
├── if(mbDeactivateLocalizationMode)
├──────────────── mpTracker->InformOnlyTracking(false);  // 取消純定位模式
│
├── if(mbReset)  // 檢測是否需要重新啟動
├────────── mpTracker->Reset();  // 重新啟動
│
├── Tcw = mpTracker->GrabImageStereo(imLeft,imRight,timestamp);  // 執行二元跟踪
```

透過上述流程可發現輸入數據後，會執行3個步驟 : 
1. 判斷是否開啟純定位模式
2. 判斷是否需要重新啟動Tracking
3. 執行二元跟踪程序，GrabImageStereo有3個參數，分別為左目、右目和時間戳

## 深度相機入口 TrackRGBD
``` 
TrackRGBD
│
├── if(mbActivateLocalizationMode)  // 檢測是否開啟純定位模式
├──────────────── mpLocalMapper->RequestStop();  // 停止局部地圖建構
├──────────────── mpTracker->InformOnlyTracking(true);  // 設置成純定位模式
│
├── if(mbDeactivateLocalizationMode)
├──────────────── mpTracker->InformOnlyTracking(false);  // 取消純定位模式
│
├── if(mbReset)  // 檢測是否需要重新啟動
├────────── mpTracker->Reset();  // 重新啟動
│
├── Tcw = mpTracker->GrabImageRGBD(im,depthmap,timestamp);  // 執行RGBD跟踪
```

整體流程與二元相同，不同在最後的入口為RGBD跟踪，輸入參數為像素圖、深度圖與時間戳

## 一元相機入口 TrackMonocular
```
TrackMonocular
│
├── if(mbActivateLocalizationMode)  // 檢測是否開啟純定位模式
├──────────────── mpLocalMapper->RequestStop();  // 停止局部地圖建構
├──────────────── mpTracker->InformOnlyTracking(true);  // 設置成純定位模式
│
├── if(mbDeactivateLocalizationMode)
├──────────────── mpTracker->InformOnlyTracking(false);  // 取消純定位模式
│
├── if(mbReset)  // 檢測是否需要重新啟動
├────────── mpTracker->Reset();  // 重新啟動
│
├── Tcw = mpTracker->GrabImageMonocular(im,timestamp);  // 執行一元跟踪
```

整體流程也與二元相同，不同在最後的入口為一元跟踪，輸入參數為像素圖與時間戳