---
title: "NTU - 機器學習 Week 5 - Normalization"
date: 2021-04-11T15:34:25+08:00
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

![cover 2](/2021NTUML/Week5/cover-2.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video](https://www.youtube.com/watch?v=BABPWOkSbLE)

Youtube Video Link (English): [Video](https://www.youtube.com/watch?v=t3u3WshJQV8)

## Changing Landscape
![Changing Landscape](/2021NTUML/Week5/change.JPG)
如果有個簡易的模型，x1輸入值小，因此當w1改變時，loss也會有微小的改變，但x2因為輸入大，所以對loss的改變也會很大，這使error surface變成橢圓形。因為兩參數對loss斜率差很大，導致error surface相當難訓練，因此要使用`Batch Normalization`來使error surface變比較好訓練

## Feature Normalization
![Changing Landscape](/2021NTUML/Week5/normalization.JPG)
透過這種Normalization，最後得到平均為0，且變異數是1，因此數值分布會在0左右，所以做出比較好得error surface，做梯度下降時，能收斂得快一些

![Considering Deep Learning](/2021NTUML/Week5/considerDL.JPG)
將Normalization後得數值丟入network做計算，進一步思考會發現只有W1輸入的值有Normalize，但W2沒有，那如果z得數值差異大那對W2的訓練會不會變困難?因此輸入W2的值也應該要做Normalize

Q : 那要在activation function前做Normalize還是之後做?

A : 實作上其實差異不大，如果activation function是使用sigmoid的話建議在之前做

![Considering Deep Learning 2](/2021NTUML/Week5/normalization-1.JPG)
假設現在對z做Normalize

![Considering Deep Learning 3](/2021NTUML/Week5/considerDL-1.JPG)
由於z、mean與std都是向量，因此這裡的除法是對向量中的每個對應的元素計算。由於mean與std是經過z1、z2與z3計算得到的，沒做Feature Normalization時，各個是獨立計算的，z1只影響a1，但當Feature Normalization後，z1改變時，會對影響到a2與a3。因此現在的network不是分別處理，而是一次處理多個輸入。由於GPU記憶體有限，因此network不會一次考慮整個訓練資料，而是考慮一個batch，這就是`Batch Normalization`，但這適用於batch size較大的情況，這樣計算batch的mean與std比較能得到資料的分布

![Batch Normalization](/2021NTUML/Week5/batch.JPG)
有時Batch Normalization還加入兩個未知參數來做訓練，這兩參數需要透過模型另外再學習出來，那為何需要加入這兩未知參數呢?由於做完Batch Normalization時，平均值為0對network多了限制，因此透過這兩參數來調整

![Batch Normalization Testing](/2021NTUML/Week5/batch_test.JPG)
有Batch Normalization後要怎麼測試呢?一般會想說一次測試一組batch的資料，但實際上不會等訓練出一組batch再測試，如pytorch會使用`moving average`來將訓練的mean與std進行處理，測試時就不用再計算batch的mean與std，而是使用moving average後的數值來測試

![Batch Normalization](/2021NTUML/Week5/batch-nor.JPG)
上圖是Batch Normalization論文中的圖表，可以發現有做Batch Normalization的模型可以使用較短的時間達到終點

![Internal Covariate Shift](/2021NTUML/Week5/internal.JPG)
原始Batch Normalization提出`Internal Covariate Shift`來解釋為何Batch Normalization能優化，簡單來說是更新參數時，更新後的參數是對更新前的參數比較好不是對更新後的，因此做Batch Normalization能先使原始參數的分布較接近更新後的，使更新參數的方向往適合更新後參數的方向走，但仍然有其他論文做實驗來解釋其他理由

![Internal Covariate Shift 1](/2021NTUML/Week5/internal-1.JPG)
上圖所描述文章講解透過實驗與理論來證實Batch Normalization能改變error surface，使error surface變得比較不崎嶇。要改變error surface其實還有其他方法，試其他方法改變error surface後發現與Batch Normalization的表現差不多，因此感嘆說Batch Normalization是偶然發現的，但是是一種有用的方法

## To Learn More
* Batch Renormalization : [Link](https://arxiv.org/abs/1702.03275)
* Layer Normalization : [Link](https://arxiv.org/abs/1607.06450)
* Instance Normalization : [Link](https://arxiv.org/abs/1607.08022)
* Group Normalization : [Link](https://arxiv.org/abs/1803.08494)
* Weight Normalization : [Link](https://arxiv.org/abs/1602.07868)
* Spectrum Normalization : [Link](https://arxiv.org/abs/1705.10941)

