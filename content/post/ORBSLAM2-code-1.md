---
title: "ORBSLAM2 程式碼解析 - 圖片輸入"
date: 2021-04-17T00:46:46+08:00
draft: false
toc: true
comment: true

categories:
  - ORBSLAM2 程式碼解析
tags:
  - ORBSLAM
  - code analysis
---

* ORB-SLAM2: an Open-Source SLAM System for Monocular, Stereo and RGB-D Cameras
(https://arxiv.org/abs/1610.06475)
* ORBSLAM2 : https://github.com/raulmur/ORB_SLAM2
* ORBSLAM2 for Windows : https://github.com/phdsky/ORBSLAM24Windows 

## System Overview
![System Overview](/ORBSLAM2-img/system-overview.JPG)

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

## 圖片輸入
![System Input](/ORBSLAM2-img/system_input.JPG)
* 註 : `mpIniORBextractor`比`mpORBextractorLeft`多提取一倍的特徵點

### Stereo
```C++
cv::Mat Tracking::GrabImageStereo(const cv::Mat &imRectLeft, const cv::Mat &imRectRight, const double &timestamp)
{
    // 輸入圖像
    mImGray = imRectLeft;
    cv::Mat imGrayRight = imRectRight;

    // 轉為灰階圖
    if(mImGray.channels()==3)
    {
        if(mbRGB)
        {
            cvtColor(mImGray,mImGray,CV_RGB2GRAY);
            cvtColor(imGrayRight,imGrayRight,CV_RGB2GRAY);
        }
        else
        {
            cvtColor(mImGray,mImGray,CV_BGR2GRAY);
            cvtColor(imGrayRight,imGrayRight,CV_BGR2GRAY);
        }
    }
    else if(mImGray.channels()==4)
    {
        if(mbRGB)
        {
            cvtColor(mImGray,mImGray,CV_RGBA2GRAY);
            cvtColor(imGrayRight,imGrayRight,CV_RGBA2GRAY);
        }
        else
        {
            cvtColor(mImGray,mImGray,CV_BGRA2GRAY);
            cvtColor(imGrayRight,imGrayRight,CV_BGRA2GRAY);
        }
    }

    // 建構Frame
    mCurrentFrame = Frame(mImGray,imGrayRight,timestamp,mpORBextractorLeft,mpORBextractorRight,mpORBVocabulary,mK,mDistCoef,mbf,mThDepth);

    // Track
    Track();

    return mCurrentFrame.mTcw.clone();
}
```

### RGBD
```C++
cv::Mat Tracking::GrabImageRGBD(const cv::Mat &imRGB,const cv::Mat &imD, const double &timestamp)
{
    // 輸入圖像
    mImGray = imRGB;
    cv::Mat imDepth = imD;

    // 轉為灰階圖
    if(mImGray.channels()==3)
    {
        if(mbRGB)
            cvtColor(mImGray,mImGray,CV_RGB2GRAY);
        else
            cvtColor(mImGray,mImGray,CV_BGR2GRAY);
    }
    else if(mImGray.channels()==4)
    {
        if(mbRGB)
            cvtColor(mImGray,mImGray,CV_RGBA2GRAY);
        else
            cvtColor(mImGray,mImGray,CV_BGRA2GRAY);
    }

    if((fabs(mDepthMapFactor-1.0f)>1e-5) || imDepth.type()!=CV_32F)
        imDepth.convertTo(imDepth,CV_32F,mDepthMapFactor);

    // 建構Frame
    mCurrentFrame = Frame(mImGray,imDepth,timestamp,mpORBextractorLeft,mpORBVocabulary,mK,mDistCoef,mbf,mThDepth);

    // Track
    Track();

    return mCurrentFrame.mTcw.clone();
}
```

### Mono
```C++
cv::Mat Tracking::GrabImageMonocular(const cv::Mat &im, const double &timestamp)
{
    // 輸入圖像
    mImGray = im;

    // 轉為灰階圖
    if(mImGray.channels()==3)
    {
        if(mbRGB)
            cvtColor(mImGray,mImGray,CV_RGB2GRAY);
        else
            cvtColor(mImGray,mImGray,CV_BGR2GRAY);
    }
    else if(mImGray.channels()==4)
    {
        if(mbRGB)
            cvtColor(mImGray,mImGray,CV_RGBA2GRAY);
        else
            cvtColor(mImGray,mImGray,CV_BGRA2GRAY);
    }

    if(mState==NOT_INITIALIZED || mState==NO_IMAGES_YET)
        mCurrentFrame = Frame(mImGray,timestamp,mpIniORBextractor,mpORBVocabulary,mK,mDistCoef,mbf,mThDepth);
    else
        mCurrentFrame = Frame(mImGray,timestamp,mpORBextractorLeft,mpORBVocabulary,mK,mDistCoef,mbf,mThDepth);

    // Track
    Track();

    return mCurrentFrame.mTcw.clone();
}
```