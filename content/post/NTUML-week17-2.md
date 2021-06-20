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

## Knowledge Distillation
![Knowledge Distillation](/2021NTUML/Week17/distillation.JPG)
接下來介紹`Knowledge Distillation`，與Network Pruning有些相似。首先訓練一個大模型(Teacher Net)，接著訓練小模型(Student Net)，小模型會輸入跟大模型相同的資料，但Label不是用資料集的，而是大模型分類好的。因此大模型判斷錯誤的，學生也會跟著錯，那為什麼要這樣做呢?與Network Pruning的理由相同，訓練一個大模型來幫助訓練小模型比直接訓練小模型來的容易，且大模型可以提供額外的資訊來幫助訓練小模型

![Knowledge Distillation 1](/2021NTUML/Week17/distillation-1.JPG)
大模型甚至可以不是一個模型，而是多個模型Ensemble組合起來的

![Knowledge Distillation 2](/2021NTUML/Week17/distillation-2.JPG)
在使用Knowledge Distillation做分類時，通常會將Softmax除以一個數值T，這個值大於1的話會使原本集中的分布變得平滑一些，因為集中的分布讓小模型學習和直接拿資料集的Label學習是差不多的，透過除以T值讓分布平滑一些，才比較能讓小模型學到各類別之間的關係

若想了解更多Knowledge Distillation可以參考以下連結 : 
* Distilling the Knowledge in a Neural Network : [Link](https://arxiv.org/pdf/1503.02531.pdf)

## Parameter Quantization
![Parameter Quantization](/2021NTUML/Week17/param.JPG)
`Parameter Quantization`用意在使用較小的空間來儲存參數，像可以降低參數的精度，使用較少的bit數來儲存參數，還有更進一步的方法是`Weight clustering`，將Weight數值接近的分成一群，那一群可以使用一個數值來代替，通常是使用一群中的平均值，這樣只需紀錄每群的數值與參數屬於哪一群就好，再更進一步的話，可以使用通訊中的技巧`Huffman encoding`，將常出現的使用較少的bit來記錄，較少出現的使用較多的bit來記錄

![Parameter Quantization 1](/2021NTUML/Week17/param-1.JPG)
最極致的就是每個參數只需要1 bit就好，若想了解如何做到Binary Weights可以參考以下連結 : 
* BinaryConnect: Training Deep Neural Networks with binary weights during propagations : [Link](https://arxiv.org/abs/1511.00363)
* Binarized Neural Networks: Training Deep Neural Networks with Weights and Activations Constrained to +1 or -1 : [Link](https://arxiv.org/abs/1602.02830)
* XNOR-Net: ImageNet Classification Using Binary Convolutional Neural Networks : [Link](https://arxiv.org/abs/1603.05279)

![Parameter Quantization 2](/2021NTUML/Week17/param-2.JPG)
有人可能很好奇使用Binary Weights會不會表現不好，上表是使用Binary Connect的方式結果比正常的方式還要好，理由是使用Binary Weights給模型較大的限制，使其不容易overfitting

## Architecture Design - Depthwise Separable Convolution 
![Depthwise Separable Convolution](/2021NTUML/Week17/depth.JPG)
首先介紹`Depthwise Convolution`，假設輸入的圖片是2個Channel，那麼Filter的數量也會是2個，每個Filter只管一個Channel，不像基本CNN，Filter數量與Channel數無關。這樣子的作法會發現每個Channel沒有互動，沒辦法抓到跨Channel的特徵

![Depthwise Separable Convolution 1](/2021NTUML/Week17/depth-1.JPG)
所以需要有`Pointwise Convolution`，這裡就會有很多Filter，但不同於一般的CNN，需限制每個Filter的大小是1*1。在上圖所需的參數量是8

![Depthwise Separable Convolution 2](/2021NTUML/Week17/depth-2.JPG)
比較一般CNN與Depthwise Pointwise Convolution可以得到兩者參數比例，通常Output channels會很大，因此關鍵在kernel size。因此假設kernel size為3，那麼從一般CNN變成Depthwise Pointwise Convolution參數量會減少到約原本的1/9

![Low rank approximation](/2021NTUML/Week17/low.JPG)
以前減少參數量的方式是在兩層之間加入一個Linear layer，若k夠小就能減少參數量，因為原本需要W個參數量，像在只需要U加V個就好，而Depthwise Pointwise Convolution也是使用這樣的概念

![Low rank approximation 1](/2021NTUML/Week17/low-1.JPG)
一般的CNN用1層來計算該區域的數值，而相同的區域若使用Depthwise Pointwise Convolution就會被拆成2層，一層是Depthwise另一層是Pointwise，用這樣的方式來減少參數量

想了解更多Architecture Design來減少參數量可以參考以下連結 : 
* SqueezeNet: AlexNet-level accuracy with 50x fewer parameters and <0.5MB model size : [Link](https://arxiv.org/abs/1602.07360)
* MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications : [Link](https://arxiv.org/abs/1704.04861)
* ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices : [Link](https://arxiv.org/abs/1707.01083)
* Xception: Deep Learning with Depthwise Separable Convolutions : [Link](https://arxiv.org/abs/1610.02357)
* GhostNet: More Features from Cheap Operations : [Link](https://arxiv.org/abs/1911.11907)

## Dynamic Computation
![Dynamic Computation](/2021NTUML/Week17/dynamic.JPG)
接下來介紹`Dynamic Computation`，與前面介紹要做的不同，這是要模型能自動調整所需的運算量，因為有時相同的模型會跑在不同的設備，不同設備會有不同的運算資源，但不希望重新訓練，希望模型能自動調整。即使在相同設備上，當電力不足時會降低運算量，所以也有可能需要這樣的技術。那為何不針對不同的情況訓練一堆模型呢?因為這不理想，且小型設備通常也無法儲存這麼多模型

![Dynamic Computation 1](/2021NTUML/Week17/dynamic-1.JPG)
第一個方法是自動調整模型深度，運算資源充足時就訓練較多層，當運算資源缺乏時就訓練較少層，但這樣通常表現不好，若想了解較好的調整深度方法可以參考以下連結 : 
* Multi-Scale Dense Networks for Resource Efficient Image Classification : [Link](https://arxiv.org/abs//1703.09844)

![Dynamic Computation 2](/2021NTUML/Week17/dynamic-2.JPG)
能調整深度，當然也能調整寬度，評量當下的運算量來決定模型要訓練多寬，但這樣表現也不佳，所以想了解較好調整寬度的方法可以參考以下連結 : 
* Slimmable Neural Networks : [Link](https://arxiv.org/abs/1812.08928)

![Dynamic Computation 3](/2021NTUML/Week17/dynamic-3.JPG)
調整寬度與深度的程度多由人類所決定，那是否能由模型視情況自行決定呢?因為有時訓練資料時，裡面有容易辨識的也有難以辨識的。想了解如何解決可以參考以下連結 : 
* SkipNet: Learning Dynamic Routing in Convolutional Networks : [Link](https://arxiv.org/abs/1711.09485)
* Dynamic Runtime Feature Map Pruning : [Link](https://arxiv.org/abs/1812.09922)
* BlockDrop: Dynamic Inference Paths in Residual Networks : [Link](https://arxiv.org/abs/1711.08393)