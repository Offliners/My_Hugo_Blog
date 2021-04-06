---
title: "NTU - 機器學習 Week 3 - Loss Function: Classification"
date: 2021-04-06T15:46:03+08:00
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

![cover 1](/2021NTUML/Week3/cover-1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video](https://www.youtube.com/watch?v=O2VkP8dJ5FE)

Youtube Video Link (English): [Video](https://www.youtube.com/watch?v=jqVONJ-Wn8w)

## Classification as Regression?
![Classification as Regression?](/2021NTUML/Week3/as.JPG)
如果把Classification當成Regression來看，將每個class用編號來記錄，那意味著class 1跟class 2比較像，class 1跟class 3比較不像，因此這方法會有點瑕疵

![One hot vector](/2021NTUML/Week3/onehot.JPG)
透過使用`one hot vector`，就不會有上述問題，因為把one hot vector算距離，class之間兩兩距離都是相同的。那現在目標需要三個數值，但以前regression的output都是一個數值，那要怎麼做呢?

![Network output](/2021NTUML/Week3/output.JPG)
那只需要將原本output一個數值的方法重複三次，乘上3個不同的weight加上bias得到三個數值，得到的這個向量期望與目標向量越接近越好

![VS](/2021NTUML/Week3/vs.JPG)
在做Classification時，往往會加上輸出向量再通過`softmax`得到另一個向量再去跟label做比較。簡單來說，因為label是0或1的值，因此通過softmax能將任何值移到0~1之間，方便來做跟label的比較

## Softmax
![Softmax](/2021NTUML/Week3/softmax.JPG)
透過上次可得知，每個y值介於0~1之間，且總和為1。這裡討論三個class的情形，那如果只有兩個class呢?也可以使用softmax，但常聽到的是sigmoid，其實兩者是等價的

## Loss of Classification
![Loss of Classification](/2021NTUML/Week3/loss.JPG)
以前計算預測值與實際值的距離可能常用MSE，但在分類問題更常用`cross-entrory`。使用pytorch時，cross-entrory與softmax是綁在一起的，使用cross-entrory時，pytorch會自動將softmax加到最後一層

![Loss of Classification](/2021NTUML/Week3/loss1.JPG)
* Mathematical proof : [Video](https://www.youtube.com/watch?v=fZAZUYEeIMg)

比較兩者的error surface，如果y1大y2小時loss小，y1小y2大時loss大，因此期望參數能走到右下角loss小的地方。假設一開始在左上角，由於在cross-entrory下左上角仍有斜率，因此能往右下走。但在MSE下，左上角非常平坦，gradient小，使用梯度下降法會卡住，有很大機率會訓練不起來，但使用Adam會調大learning rate，還是有機會走到右下角，只是訓練還是困難的。總結來說，改變loss function能改變訓練的難易度

## To Learn More
* Classification : [Video](https://youtu.be/fZAZUYEeIMg)
* Logistic Regression : [Video](https://youtu.be/hSXFuypLukA)

