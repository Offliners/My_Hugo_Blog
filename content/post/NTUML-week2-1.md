---
title: "NTU - 機器學習 Week 2 - Guideline of ML: overfit"
date: 2021-04-03T23:32:44+08:00
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

![cover1](/2021NTUML/Week2/cover1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video 1](https://www.youtube.com/watch?v=WeHM2xpYQpw) [Video 2](https://www.youtube.com/watch?v=QW6uINn7uGk)

Youtube Video Link (English): [Video 1](https://youtu.be/3qgKpBptyFY) [Video 2](https://youtu.be/yz7QS1I6omw)

## Framework of ML
![framework](/2021NTUML/Week2/framework.JPG)

## General Guide
![General Guide 1](/2021NTUML/Week2/general-guide-1.JPG)
如果覺得訓練不滿意時，首先檢查training data的loss，看模型是否學起來再檢查testing data

![Model Bias](/2021NTUML/Week2/model-bias.JPG)
有可是`model bias`使training data的loss無法變低。由於model過於簡單，透過訓練所有參數所得到的function set，但因為模型簡單使得function set小，因此這個function set沒有包含任何一個function使得loss變低，可以讓loss變低的function不在這個model可以描述的範圍裡面。簡單來說，想在大海裡撈針(使loss變低的function)，但針根本不在海裡，怎麼撈都撈不到。

解決方法 : 重新設計模型，使模型有更大的彈性，可以增加feature，也可以使用deep learning的方式

![General Guide 2](/2021NTUML/Week2/general-guide-2.JPG)
但不代表training時loss大就一定是model bias，有可能是optimization做不夠好

![Optimization Issue](/2021NTUML/Week2/optimization-issue.JPG)
這堂課都使用梯度下降法，但這方法有些問題，如可能卡在區域最小值。如上圖藍色圖形內表示model可以表示函式的集合，這集合內確實存在loss低的function，但梯度下降法沒辦法幫我們找到他。簡單來說，想大海撈針，針確實在海裡，但我們沒辦法撈起來。

![Which one](/2021NTUML/Week2/which.JPG)
當training data的loss不夠低時，是哪個問題呢?

![Compare](/2021NTUML/Week2/compare.JPG)
可以透過比較不同的模型來判斷。如果看testing data的loss表現，很多人可能覺得56層沒有比20層做得好，這是overfitting。但這不是overfitting，透過比較training data的loss，發現20層的loss比56層的低，代表56層的optimization沒有做好。這也不會是model bias的問題，56層彈性更大，一定能做到20層能做到的事，因此這就是Optimization Issue。

![Method1](/2021NTUML/Week2/method-1.JPG)
因此要知道optimization有沒有做好，可以先跑一些小的或是淺的模型，甚至用一些不是deep learning的方法，如Linear model或是SVM，他們比較容易做optimization。接著做深的model，如果深的model彈性大但loss沒辦法做得比淺的低，表示有Optimization Issue。以上次例子來看，模型5層出現了Optimization Issue。那要怎麼辦呢?下堂課會講解

![General Guide 3](/2021NTUML/Week2/general-guide-3.JPG)
如果已經可以讓training data的loss變小，那就可以看testing data的loss，當testing data的loss也小，那就訓練完成。

![General Guide 4](/2021NTUML/Week2/general-guide-4.JPG)
那當覺得testing data的loss還不夠小時，如果training data的loss小，testing data的loss大，那有可能遇到overfitting的問題。

![Overfitting-1](/2021NTUML/Week2/overfitting-1.JPG)
舉一個極端例子，假設今天有個模型，如果輸入有出現在training data中那就輸出他的值，若沒有則輸出隨機值。此模型在training data的loss為0，但在沒看過的teating data的loss變得很大

![Overfitting-2](/2021NTUML/Week2/overfitting-2.JPG)
一般情況下，假設今天實際資料呈現為二次曲線，但我們不知道，我們只能知道線上的3個點(訓練資料)，如果有個能力強的模型，彈性很大，他會通過訓練資料的三個點，但其他地方會呈現freestyle，因此丟入測試資料時會發現沒通過使loss變大

![Method2](/2021NTUML/Week2/method-2.JPG)
那要如何解決overfitting呢?方法一是增加訓練資料，也是最有效解決的方法。方法二是Data augmentation，如做影像辨識時，可以翻轉資料來增加資料，這Data augmentation不能隨便做，如翻轉圖片不會顛倒圖片，因為這可能不是真實世界會出現的影像。因此要根據對資料的理解，來選擇合適Data augmentation的方式

![Method3](/2021NTUML/Week2/method-3.JPG)
剛剛是增加資料的方式，也可以透過增加模型限制來避免overfitting。比如可以限制模型一定是二次曲線，但要如何得知呢?取決於對問題的理解。最好的是模型與資料背後產生得過程相同

![Method4](/2021NTUML/Week2/method-4.JPG)
如何限制模型呢?
* 漸少參數、神經元，或者共用參數(CNN)
* 用比較少的feature
* Early stopping
* Regularization
* Dropout

![Method5](/2021NTUML/Week2/method-5.JPG)
但也不要給模型過多的限制。假設模型限制是一條直線，雖然沒辦法找到一條同時通過這三個點，但可以找到一條線loss是最低的，但這會在testing data中得到較大的loss。那這是overfitting嗎?不是，這是model bias

![Trade off](/2021NTUML/Week2/trade-off.JPG)
以上會產生矛盾狀況，當model越複雜，但太複雜時teting的loss會暴增。所以期望能選到中庸的模型

![Kaggle](/2021NTUML/Week2/kaggle.JPG)
在kaggle上，如果是選擇在public data中loss最低的模型的話，往往在private data中表現會差很多

![Cross Validation](/2021NTUML/Week2/cross-validation.JPG)
可以將training data拆成Training set與Validation set，透過衡量Validation set的結果，比較不會發生public分數很好，但private分數很差的情況

![N-fold Cross Validation](/2021NTUML/Week2/n-cross-validation.JPG)
但如果training data分的不好，有分到結果差的資料。因此可以透過N-fold Cross Validation，將資料切成3等分，如上圖例子切成3等分，首先前兩份用來訓練，會後一份用來驗證，接著換順序，總共重複3次。接著假設有3個模型，將這三個模型在3種情況下跑出結果，再將結果取平均，假設model 1結果最好，那就將training data全丟給model 1訓練，最後再將testing data給訓練好的model 1

![General Guide 5](/2021NTUML/Week2/general-guide-5.JPG)
除了overfitting，還會有一種錯誤形式是mismatch，有些人會說mismatch是一種overfitting，但這是名詞定義問題。一般的overfitting可以透過蒐集更多的資料來克服，但mismatch是指說訓練資料與測試資料分布不同

![Mismatch](/2021NTUML/Week2/mismatch.JPG)
如在Homework 11，訓練資料與測試資料完全不同，因此增加訓練資料沒有用。因此要解決這種問題，等待homework 11再講。要判斷是否發生Mismatch，要看對資料本身的理解，要對訓練資料與測試資料有些理解才能判斷

