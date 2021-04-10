---
title: "NTU - 機器學習 Week 1 - Introduction of ML/DL"
date: 2021-04-03T17:05:34+08:00
draft: false
toc: true
comment: true

categories:
  - ML
  - DL
  - Hung Yi - Lee
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week1/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video 1](https://youtu.be/Ye018rCVvOo) [Video 2](https://youtu.be/bHcJCp2Fyxs)

Youtube Video Link (English): [Video 1](https://www.youtube.com/watch?v=Y87Ct23H3Kw) [Video 2](hhttps://www.youtube.com/watch?v=O69EqgzUl9U)

## What is Machine Learing?
Q : 什麼是機器學習?

A : 機器學習簡單來說就是`讓機器學會找一個函式的能力`
![ML](/2021NTUML/Week1/ML.JPG)
舉例來說:
1. 語音辨識(Speech Recognition) : 函式輸入是聲音訊號，輸出是這段訊號的內容
2. 影像辨識(Image Recognition) : 函式輸入是圖片，輸出是這張圖片的內容
3. Alpha Go : 函式輸入是棋盤上黑白棋的位置，輸出是機器下一步落子的位置

## Different types of Functions
![MLtype](/2021NTUML/Week1/MLtype.JPG)

1. Regression : 函是輸出為純量
    舉例 : 輸入今天的PM2.5濃度、溫度等，預測明天的PM2.5濃度
2. Classification : 給定選項，函示輸出為機器從這些選項中的選擇
    舉例 : 輸入一封電子郵件，輸出為是否為垃圾郵件

![alphaGO](/2021NTUML/Week1/AlphaGO.JPG)
Alpha Go也是一種Classification，輸入棋盤上黑白棋的位置，輸出棋盤19 * 19可以落子的位置選出下一步應該落子的位置

Q : 機器學習只有這兩個任務嗎?

A : 還有`Structured Learning`，要機器產生有結構的物件

## How to find a function?
### Case study : Youtube Viewer Prediction
![Youtube Viewer](/2021NTUML/Week1/yt.JPG)
透過輸入Youtube後台數據(2017~2020觀看流量)，預測隔天觀看數

### 1. Function with Unknown Parameter 
![ML step1](/2021NTUML/Week1/MLStep1.JPG)
透過線性模型$$y = b + wx_1$$將輸入乘上weight(w)加上bias(b)，w和b皆為不知道的參數

### 2. Define Loss from Training Data
![ML step2](/2021NTUML/Week1/MLStep2.JPG)
透過輸入資料到線性模型來輸出預測值，並將預測值與label相減取絕對值來計算誤差，label為正確數值

![ML step2-1](/2021NTUML/Week1/MLStep2-1.JPG)
剛剛例子使用MAE，要選擇使用MAE或MSE看任務的需求以及對任務的理解。如果輸出與預測都是機率的話，會選擇Cross-entropy

![ML step2-2](/2021NTUML/Week1/MLStep2-2.JPG)
上圖如果越偏紅色系表示誤差越大，藍色系誤差越小。這樣的等高線圖為`Error Surface`

### 3. Optimization
![ML step3](/2021NTUML/Week1/MLStep3.JPG)
使用梯度下降法(Gradient Descent)，首先隨機選擇一個初始值，接著對loss function對w取微分並帶入初始值，如果是負值則增加w，反之正值則減少w，一直更新參數w。除了w之外還有η(learning rate)，需自行設定，屬於機器學習中超參數的部分。

使用梯度下降法，容易產生區域最小值(Local minima)的問題，但實際使用這方法，最大的難題不是這個，之後會再提。

![ML step3-1](/2021NTUML/Week1/MLStep3-1.JPG)
之前是對單一參數w做梯度下降法，接著是對兩參數w與b

![ML step3-2](/2021NTUML/Week1/MLStep3-2.JPG)
再來對線性模型使用梯度下降法算w與b，Error Surface呈現如上

### Performance
![Performance](/2021NTUML/Week1/performance.JPG)
結果分析:
紅色現為真實觀看數，藍色線為機器預測，藍色線幾乎為紅色線往右平移一天，因為輸入資料是前一天的觀看人次，因此機器幾乎是拿前一天觀看人次做預測


## How to Improve Performance?
### 1. More Feature
![Improve 1](/2021NTUML/Week1/improve1.JPG)
輸入前7天的資料，loss從0.48k降到0.38k

輸入前28天的資料，loss從0.48k降到0.33k

輸入前56天的資料，loss從0.48k降到0.32k

看來考慮更多天沒辦法再更進步了。

### 2. Flexible Model
![Improve 2-1](/2021NTUML/Week1/improve2-1.JPG)
由於Linear model過於簡單，被Model bias所限制過，因此需要一個更複雜更有彈性的模型

![Improve 2-2](/2021NTUML/Week1/improve2-2.JPG)
透過`acitvation function`來使模型更有彈性，上圖使用`sigmoid`來減少model bias

![Improve 2-3](/2021NTUML/Week1/improve2-3.JPG)
可以透過足夠多個acitvation function，可以變成任何扭曲的曲線

![Improve 2-4](/2021NTUML/Week1/improve2-4.JPG)
再結合第一點，輸入更多feature

## Redefine Model Training
### 1. New Function with Unknown Parameter
![new-model1](/2021NTUML/Week1/new-model1.JPG)
將複雜的運算簡單化

![new-model2](/2021NTUML/Week1/new-model2.JPG)
使用矩陣的方式來表達數學式

![new-model3](/2021NTUML/Week1/new-model3.JPG)
σ表示acitvation function，這裡的acitvation function是使用sigmoid function，所以以上是將輸入x經過w縮放和b的位移，再通過acitvation function，通過後再經過c縮放和b的位移。兩個b不相同，一個是向量，另一個是純量。

### 2. Redefine Loss from Training Data
![new-model4](/2021NTUML/Week1/new-model4.JPG)
與前面相同，只是function不同而已

### 3. Optimization of New Model
![new-model5](/2021NTUML/Week1/new-model5.JPG)
也與前面相同，使用梯度下降法，首先隨機選取初始值，接著把L對每個未知參數取微分，所得值及合成一個向量，這個向量就是Gradient。

![new-model6](/2021NTUML/Week1/new-model6.JPG)
再來就是透過learning rate來做參數更新

![new-model7](/2021NTUML/Week1/new-model7.JPG)
實際上，會將一大筆資料分成數個Batch，每個Batch都更新好參數後就是一個epoch。Update是更新參數，epoch是更新好每個Batch的參數。Batch size也是模型的超參數。

### Why Sigmoid?
![why1](/2021NTUML/Week1/why1.JPG)
為何使用sigmoid?也可以使用ReLU，使用兩個ReLU可以組合成一個Hard Sigmoid。

![why2](/2021NTUML/Week1/why2.JPG)
至於哪個比較好，之後會說。

### Experimental Results
![exp1](/2021NTUML/Week1/exp1.JPG)
100個ReLU有顯著差別，訓練Loss從0.32k降到0.28k

1000個ReLU雖然訓練loss有降低，但沒看過的測試資料就沒有差別

## What else can we do?
![layer](/2021NTUML/Week1/layer.JPG)
可以透過增加層數，先前是經過一次運算，而現在是將一次計算結果再經過多次計算。層數也是模型超參數之一

### Experimental Results
![exp2](/2021NTUML/Week1/exp2.JPG)
實驗結果顯示3層可以將訓練loss降到0.14k，沒看過的測試資料可以降到0.38k

![exp3](/2021NTUML/Week1/exp3.JPG)
可以發現機器預測更接近實際值，至於問號的地方是機器高估，由於那天過年，機器不知道新年，所以預測較差

## Deep Learning
![DL](/2021NTUML/Week1/DL.JPG)
為何不排一排多個activation function使模型變肥，而是加深模型，日後再說

## Why don't we go deeper?
![exp4](/2021NTUML/Week1/exp4.JPG)
加深到第四層時，訓練資料有變好，但沒看過的測試資料反而變差了，這就是overfitting



