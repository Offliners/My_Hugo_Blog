---
title: "NTU - 機器學習 Week 2 - Tips for Training: Batch and Momentum"
date: 2021-04-05T21:55:10+08:00
draft: false
toc: true

categories:
  - ML
  - DL
  - Hung Yi - Lee
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover 3](/2021NTUML/Week2/cover3.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video](https://www.youtube.com/watch?v=zzbr1h9sF54)

Youtube Video Link (English): [Video](https://youtu.be/MNoEQ9w-AbE)

## Optimization with Batch
![Batch](/2021NTUML/Week2/batch.JPG)
每次update參數是將每個Batch出來計算loss與gradient，將每個batch計算過後是一個epoch。每個epoch都會經過shuffle，因此每個epoch中batch的資料都不一樣

## Small Batch v.s. Large Batch
![Small Batch v.s. Large Batch 1](/2021NTUML/Week2/vs3.JPG)
`Large Batch` 須看完每筆資料才更新參數，因此更新時間長，但每次更新很平穩

`Small Batch` 每看完一筆資料就更新參數，因此更新時間短，但每次更新會顯得不穩

但考慮平行運算的話，Large Batch時間不一定比Small Batch長

![Parallel Computing 1](/2021NTUML/Week2/parallel1.JPG)
由上圖可知batch從1~1000所花時間差不多，但GPU仍有極限，過大的batch size仍會需要花數倍的時間

![Parallel Computing 2](/2021NTUML/Week2/parallel2.JPG)
如果batch size等於1，那60000筆資料需要update參數60000次。batch size等於1000，那60000筆資料需要update參數60次。兩個每次update的時間差不多，但跑完一個epoch的差距就很大。因此考慮平行運算後，大batch size反而比較有效率

![Small Batch v.s. Large Batch 2](/2021NTUML/Week2/vs4.JPG)
考慮平行運算後，大batch沒有時間劣勢了，且update參數比較穩定，那大batch比較好嗎?小batch比較差?

![Small Batch v.s. Large Batch 3](/2021NTUML/Week2/vs5.JPG)
雖然在小batch size下update參數比較不穩，但有noisy的gradient反而能幫助訓練。比較上面兩資料集，小batch size的準確率比較高。那這是什麼問題呢?這是`Optimization Issue`

![Small Batch v.s. Large Batch 4](/2021NTUML/Week2/vs6.JPG)
那為何小batch size沒有Optimization Issue呢?如果使用full batch size的話，走到local minima時就會停止更新參數。但如果使用small batch size的話，可能其中一個batch卡住了，但其他batch可能沒有。因此這種noisy的update參數反而對訓練比較有幫助

![Small Batch v.s. Large Batch 5](/2021NTUML/Week2/vs7.JPG)
訓練上small batch size比較好，那測試呢?透過以上的實驗，將大batch和小batch訓練到差不多好，那比對測試時，發現小batch在測試集上表現比較好。大batch在訓練好，測試差，表示發生了overfitting。那為何會這樣呢?

![Small Batch v.s. Large Batch 6](/2021NTUML/Week2/vs8.JPG)
假設有一個loss，有許多local minima，但local minima有分好壞，那怎麼區分好壞呢?如果local minima落在峽谷中(上圖右側)，會認為是壞minima;反之，落在平原中會是好的minima。會這樣區分是因為，測試與訓練的loss或有些差別，平原型的minima差異較小，峽谷型的minima會有較大的loss差異。而小batch因為update參數的方向都不同，容易跳出峽谷的loss，容易落在平原型的minima;對大batch來說，反而容易落在峽谷的minima

![Small Batch v.s. Large Batch 7](/2021NTUML/Week2/vs9.JPG)
那有沒有辦法僵固兩者的優點呢?以下的論文有探討
* Large Batch Optimization for Deep Learning: Training BERT in 76 minutes (https://arxiv.org/abs/1904.00962)
* Extremely Large Minibatch SGD: Training ResNet-50 on ImageNet in 15 Minutes (https://arxiv.org/abs/1711.04325)
* Stochastic Weight Averaging in Parallel: Large-Batch Training That Generalizes Well (https://arxiv.org/abs/2001.02312)
* Large Batch Training of Convolutional Networks
(https://arxiv.org/abs/1708.03888)
* Accurate, large minibatch sgd: Training imagenet in 1 hour
(https://arxiv.org/abs/1706.02677)

## Momentum
一個可能可以對抗local minima或saddle point的技術

![Momentum](/2021NTUML/Week2/momentum.JPG)
如果假設update參數像是把球向低處滾動，如果沒有動量的話，遇到critical point時gradient為0，那球就不再動。但如果有動量介入，球會順著慣性，有可能逃離critical point

### Vanilla Gradient Descent
![Vanilla Gradient Descent](/2021NTUML/Week2/gd.JPG)
每次更新參數會加上gradient的反方向

### Gradient Descent + Momentum
![Gradient Descent + Momentum](/2021NTUML/Week2/gdm.JPG)
加上Momentum後，除了加上gradient的反方向，還會加上前一次更新參數的方向
$$m^{0} = 0$$
$$m^{1} = -{\eta}g^{0}$$
$$m^{2} = -{\lambda}{\eta}g^{0} - {\eta}g^{1}$$
已此類推，可以發現每次加上的momentum是過去所有gradient的總和

![Gradient Descent + Momentum 1](/2021NTUML/Week2/momentum1.JPG)
透過上圖可以得知走到local minima時，雖然gradient為0(紅色箭頭)，但仍然有momentum可以逃離。即使到local maxima，gradient會讓球往左側走，但momentum夠大的話，就可以向右走找更低的loss

### Concluding Remarks
![Concluding Remarks](/2021NTUML/Week2/concluding.JPG)
