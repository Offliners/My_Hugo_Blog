---
title: "NTU Machine Learning Week 17 - Network Compression"
date: 2021-06-19T14:45:47+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 17 - Network Compression

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week17/cover-2.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/utk3EnAUh-g) [`Video 2`](https://youtu.be/xrlbLPaq_Og)

Youtube Video Link (English): [`Video 1`](https://youtu.be/CB0a3aBwND8) [`Video 2`](https://youtu.be/mGRdOGdOZ-4) 

## Why we need network compression
![why](/2021NTUML/Week17/why.JPG)
為什麼需要將大模型簡化成小模型並表現差不多呢?因為很多時候需要將模型訓練在資源有限的環境下，如智能手錶、IoT裝置等，那為什麼不將資料傳回雲端訓練再傳回去呢?常見的理由是為了低延遲與隱私

## Network Pruning
![Network Pruning](/2021NTUML/Week17/prune.JPG)
顧名思義就是修剪模型中的參數，因為大模型中不一定每個參數都有運算到，所以找出無用的參數再修剪掉

![Network Pruning 1](/2021NTUML/Week17/prune-1.JPG)
首先訓練一個大的模型，再去評估每個參數的重要性，例如weight可以使用絕對值來看其大小，Neuron可以判斷輸出不為0的次數等，評估後再修剪一些參數，通常修剪後正確率會掉一些，為了能回升回來通常會Fine-tune，Fine-tune後還能再評估個參數的重要性，這步驟可以執行多次值到模型你覺得夠小為止。那為何不一次修剪多個無用的參數呢?因為根據文獻，一次修剪過多會對模型造成極大損傷，可能無法復原回來

![Weight Pruning 1](/2021NTUML/Week17/weight.JPG)
先剪可以用Weight為單位或者用Neuron，若以Weight為單位的話，將無用的Weight去掉後形狀會變成不規則(如上右圖)，會變得不好實作，即使時做出來也會不好加速運算，因為加速運算是透過矩陣運算來實現，不規則形狀會使得運算難以平行化，若為了維持形狀可能會將值變成0，但這樣模型並沒有變小

![Weight Pruning 2](/2021NTUML/Week17/weight-2.JPG)
根據文獻可以發現即使修剪了95%左右的參數，結果在多數情況下訓練沒有加速

![Neuron Pruning 1](/2021NTUML/Week17/neuron.JPG)
因此使用Neuron Pruning會比較好實作，因為修剪掉Neuron仍可以維持好模型的形狀

![Why Pruning 1](/2021NTUML/Week17/why-1.JPG)
那有人可能問大模型與小模型有差不多的表現，那為什麼不直接訓練小模型就好呢?訓練大模型再Pruning變小太麻煩。理由是因為大模型比較好訓練，訓練小模型表現往往不好

若想了解為何大模型比較好訓練可以參考以下連結 : 
* Geometry of Loss Surfaces (Conjecture) : [Video](https://www.youtube.com/watch?v=_VuWvQUMQVk)
* The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks : [Link](https://arxiv.org/abs/1803.03635)

![Lottery Ticket Hypothesis](/2021NTUML/Week17/lottery.JPG)
接下透過大樂透假說來解釋為何大模型比較好訓練，由於模型的表現與Initial weight有關，若初始點好可能可以訓練一組好的參數。將大模型看成多個小模型的組合，透過訓練多個小模型來提高取得好參數的機率，只要一個小模型表現好，那麼大模型就會好

![Lottery Ticket Hypothesis 1](/2021NTUML/Week17/lottery-1.JPG)
大樂透假說的證實方式與Pruning有關，將大模型初始化參數後進行訓練，訓練後進行Pruning變成小模型，小模型若再重新初始化一次參數再訓練會發現訓練不起來，但小模型使用大模型的初始化參數卻可以成功訓練。因此使用大樂透假說來解釋，這個Pruning後的模型就是大模型中多個小模型裡表現最好的小模型，若只訓練小模型運氣不好的話就訓練不起來

![Lottery Ticket Hypothesis 2](/2021NTUML/Week17/lottery-2.JPG)
大樂透假說提出後也有許多相關的研究，如上圖有人使用多種Pruning的方法來實驗，發現Pruning前與Pruning後的絕對值差越大越有效。另一個有趣的結果是訓練小模型的初始參數只要與大模型初始參數有相同的正負號就能訓練起來，不論值的大小。最後一個神奇的發現是雖然是給與隨機的參數，但在大模型中已經存在一個小模型，只要套用這組隨機參數就能有好表現了

若想了解更多可以參考以下連結 : 
* Deconstructing Lottery Tickets: Zeros, Signs, and the Supermask : [Link](https://arxiv.org/abs/1905.01067)
* Weight Agnostic Neural Networks : [Link](https://arxiv.org/abs/1906.04358)

![Lottery Ticket Hypothesis 3](/2021NTUML/Week17/lottery-3.JPG)
那大樂透假說是對的嗎?不一定，有些人也提出反駁大樂透假說的論文，如上圖有人將小模型使用隨機參數來訓練，雖然表現較差(Scratch-E)，但再多訓練幾個epoch發現表現變更好(Scratch-B)。因此大樂透假說是真是假，仍尚待證實

若想了解更多可以參考以下連結 : 
* Rethinking the Value of Network Pruning : [Link](https://arxiv.org/abs/1810.05270)