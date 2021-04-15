---
title: "NTU - 機器學習 Week 7 - GAN"
date: 2021-04-15T20:04:48+08:00
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

![cover 2](/2021NTUML/Week7/cover-2.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video 1](https://youtu.be/4OWp0wDu6Xw) [Video 2](https://youtu.be/jNY1WBb8l4U)

Youtube Video Link (English): [Video 1] [Video 2]

## Network as Generator
![Network as Generator](/2021NTUML/Week7/generator.JPG)
現在要將network當成Generator使用，這個network會輸入x與隨機變數z來生成y，每次使用這個network時會透過簡單的分布來生成不同的z，這種簡單的分布可以是Gaussian Distribution或者是Uniform Distribution等。加入隨機變數z後，network輸出會變成複雜的分布。這種特殊的network叫做`Generator`

那為什麼要訓練Generator?為什麼要訓練這種輸出是一個複雜分布的模型?

## Why Distribution?
![Why Distribution?](/2021NTUML/Week7/dis.JPG)
假設今天有個任務，是要去預測之後的遊戲畫面，那可能會將過去遊戲畫面作為模型輸入，然後模型會輸出下一個預測的遊戲畫面

![Why Distribution? 1](/2021NTUML/Week7/dis-1.JPG)
但這樣會有個問題，就是有時候會發現小精靈會分裂，甚至消失。因為在訓練資料中有時候小精靈是向左轉，有時候是向右轉，那會造成機器覺得向左轉是對的，向右轉也是對的，但同時向左向右就是錯的，那要怎麼處理這類問題呢?讓機器輸出是有機率的，而不是單一的輸出

![Why Distribution? 2](/2021NTUML/Week7/dis-2.JPG)
將給network一個機率分布z時，模型輸出就會變成一個Distribution，這個Distribution包含向左轉與向右轉的可能

![Why Distribution? 3](/2021NTUML/Week7/dis-3.JPG)
那什麼時候需要這種network?當任務需要一些創造力的時候，也就是同樣的輸入卻有多種可能的輸出，而這些輸出都是對的。例如繪畫或者聊天機器人

## Generative Adversarial Network
![Generative Adversarial Network](/2021NTUML/Week7/gan.JPG)
GAN有多種的變形，多到許多不同GAN的變形卻有相同的名字，可以到以下連結去查看:

* The GAN Zoo : [Link](https://github.com/hindupuravinash/the-gan-zoo)

### Anime Face Generation
![Anime Face Generation](/2021NTUML/Week7/anime.JPG)
接著要舉的例子是生成動畫人物的臉，使用的是`Unconditional Generation`，也就是模型輸入只有Z，沒有x。因此Generator只輸入Z輸出y，然後Z是使用Normal distribution中sample出來的向量，向量維度是自訂的。所以透過輸入低維度的隨機向量，去透過Generator產生圖片這種高維度的向量(如果圖片大小是64 * 64且是RGB，那輸出的向量就會是64 * 64 * 3)

### Discriminator
![Discriminator](/2021NTUML/Week7/discriminator.JPG)
在GAN中，除了訓練Generator，還要訓練Discriminator，圖片會輸入到Discriminator，接著輸出一個數值，數值越大表示這張圖片越像是二次元人物的圖像

### Basic Idea of GAN
![Basic Idea of GAN](/2021NTUML/Week7/basic.JPG)
首先第一代的Generator因為參數是隨機的，因此產生的圖片就是雜訊，那換Discriminator來分辨這些圖片與真正的圖片不同，一開始Discriminator可能只判斷是否有眼睛，有眼睛是真正的圖片，沒有的是Generator產生的。那換第二代的Generator，調整參數來騙過Discriminator，因為Discriminator判斷的依據是是否有眼睛，所以第二代的Generator開始產生眼睛，可以騙過第一代的Discriminator，但Discriminator也會進化，第二代的Discriminator可能發現可以根據是否有頭髮或嘴巴來分辨圖片。那第三代的Generator就會繼續想辦法去騙過第二代的Discriminator。以此類推，所以Generator產生的圖片會越像二次元人物，而Discriminator也會越來越嚴苛

### Algorithm
![Algorithm](/2021NTUML/Week7/algorithm.JPG)
接下來從演算法來介紹，Generator與Discriminator是兩個Network，首先要初始化參數，再來會將Generator定住，只訓練Discriminator。由於Generator參數是隨機的又被定住，所以輸出會是雜訊，接著從database中拿出二次元人物的圖像與Generator產生的圖像來訓練Discriminator做分辨，實作上可能將真正圖片的label設為1，Generator產生的設為0，最後就是要當成分類問題或這Refression問題，如果是分類問題就是分辨0或者1，如果是Regression則是看到真正的圖片輸出1，另一些圖片輸出0

![Algorithm 1](/2021NTUML/Week7/algorithm-1.JPG)
訓練完Discriminator後，定住Discriminator改訓練Generator。為了能欺騙Discriminator，Generator輸入隨機分布的向量後產生一張圖片給Discriminator，Discriminator會給這圖片分數，因此Generator訓練的目標是Discriminator的分數越高越好。實際操作會是將Generator與Discriminator接起來變成一個大network，這個大network輸入是向量，輸出是數值，但要注意只會更新Generator的參數而已

![Algorithm 2](/2021NTUML/Week7/algorithm-2.JPG)
再來就是反覆以上的步驟

### Anime Face Generation Result
![Anime Face Generation Result](/2021NTUML/Week7/result.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-1.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-2.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-3.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-4.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-5.JPG)
![Anime Face Generation Result 1](/2021NTUML/Week7/result-6.JPG)
隨著update次數增加，圖像呈現越來越好

### StyleGAN
![StyleGAN](/2021NTUML/Week7/styleGAN.gif)

如果使用`StyleGAN`可以做得更精美

### Progressive GAN
![Progressive GAN](/2021NTUML/Week7/ProgressiveGAN.JPG)
除了二次元人物圖像，甚至連真人的人臉都可以用GAN產生，以上人臉皆為`Progressive GAN`產生

![Progressive GAN 1](/2021NTUML/Week7/ProgressiveGAN-1.JPG)
由於GAN的Generator是輸入向量得到圖片的，那可以透過輸入兩個不同的向量，並在這兩項量做內插，可以發現之間連續的變化，甚至可以由男變女、由向左看變向右看

## Theory behind GAN
![Our object](/2021NTUML/Week7/object-1.JPG)
在Generator中訓練的目標是什麼?要Minimize或Maximize什麼東西?將原本簡單分布的向量輸入Generator後會產生複雜分布的向量，那預期它與真實資料分布的情形越接近越好，那問題就在如何計算這個Divergence呢?使用KL Divergence或JS Divergence會是相當複雜的計算甚至無法計算得知

![Sample](/2021NTUML/Week7/sample.JPG)
但GAN只要能從輸入的簡易分布與輸出的複雜分布中提取資料出來，即使不知道這些分布實際長什麼樣子，只要能提取就能算Divergence

![Sample 1](/2021NTUML/Week7/sample-1.JPG)
那要GAN怎麼在只要做sample的前提下就計算出Divergence呢?這需要靠Discriminator的力量，Divergence訓練目標是看到真實資料給與較高的分數，看到生成的圖片給與較低的分數。而這可以寫成`Objective Function`，透過此式可以知道希望能Maximize V(G, D)，也就是讓D(y)越大越好。事實上這個Objective Function就是一個cross-entropy，只是多了負號，而Maximize negative cross-entropy等同於Minimize cross-entropy，代表訓練Discriminator就是在訓練一個Classifier

神奇的是，找到一個D使Objective Function最大這件事與JS Divergence有關，詳細推導可以參考以下連結 : 
* Generative Adversarial Networks : [Link](https://arxiv.org/abs/1406.2661)

![Sample 2](/2021NTUML/Week7/sample-2.JPG)
直觀上來看，假設Divergence很小，表示隨機的分布與實際的分布很像，那Discriminator就沒辦法輕易分開這兩個，因此使Objective Function的最大值會比較小。反之，Divergence很大，Discriminator能輕易分開，那Objective Function的最大值會比較大

![Sample 3](/2021NTUML/Week7/sample-3.JPG)
在Generator中卡在不知道怎麼計算Divergence，但訓練Discriminator後得到Objective Function的最大值，而這個值與Divergence有關，那就讓Discriminator的Objective Function套用到Generator。透過Generator與Discriminator互相欺騙的行為，就可以解這個MinMax的問題

![Table](/2021NTUML/Week7/table.JPG)
那為何是JS Divergence? 事實上可以透過設計不同的Objective Function，就可以量不同的Divergence，想了解如何設計不同的Objective Function獲得不同的Divergence可以參考以下連結:

* f-GAN: Training Generative Neural Samplers using Variational Divergence Minimization : [Link](https://arxiv.org/abs/1606.00709)

## Tips for GAN

