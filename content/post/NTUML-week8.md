---
title: "NTU - 機器學習 Week 8 - BERT"
date: 2021-04-21T21:33:28+08:00
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

![cover](/2021NTUML/Week8/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/e422eloJ0W4) [`Video 2`](https://youtu.be/gh0hewYkjgo) [`Video 3`](https://youtu.be/ExXA05i8DEQ)

Youtube Video Link (English): [`Video 1`] [`Video 2`] [`Video 3`]

## Model Introduction
![Name](/2021NTUML/Week8/name.JPG)
`Self-Supervised Learning`的模型命名皆來自芝麻街的角色，剩下Cookie Monster(藍)還尚有模型以此命名

![BERT](/2021NTUML/Week8/BERT.JPG)
說到Self-Supervised Learning就要提一下2018年出現的BERT模型，他是個超巨型的Transformer，有340 Million個參數，是一般Transformer參數量的3000多倍

![Model](/2021NTUML/Week8/Model.JPG)
直到現今還有更多參數量更大的模型出現

![Model 1](/2021NTUML/Week8/Model-1.JPG)
![Model 2](/2021NTUML/Week8/Model-2.JPG)
![Model 3](/2021NTUML/Week8/Model-3.JPG)
|Model|Parameter|
|-|-|
|ELMO|94M|
|BERT|340M|
|GPT-2|1542M|
|Megatron|8B|
|T5|11B|
|Turing NLG|17B|
|GPT-2|175B|
|Switch Transformer|1.6T|

## Self-supervised Learning
![Self-supervised Learning](/2021NTUML/Week8/self.JPG)
之前介紹的幾乎都是Supervised Learning，輸入的資料都是有label的。那Self-supervised Learning? 輸入的資料沒有label，所以會將輸入資料分成兩部分，一部分作為輸入，另一部分作為學習label用。有些人會稱這種沒有label的資料作為輸入的方法為Unsupervised Learning，但有人覺得Unsupervised方法涵蓋太多，因此這種自我學習label的方法改叫Self-supervised Learning

![Masking Input](/2021NTUML/Week8/mask.JPG)
那實際上是如何自己產生label? 舉BERT為例，BERT是一種Transformer的encoder，輸入一排向量，輸出一排與輸入相同長度的一排向量，然後會將輸入的向量隨機遮蔽，那要怎麼遮蔽呢? 有兩種做法，一種是換成特殊符號，另一種是隨機互換成另一個向量。然後將被遮蔽的輸入所得到的輸出經過Linear轉換再經過Softmax得到一個分布。這種方法叫`Masking`

![Masking Input 1](/2021NTUML/Week8/mask-1.JPG)
那學習的目標是什麼? 目標是原本被遮蔽的向量，也就是個分類問題，訓練時需要訓練BERT與Linear Model

![Next Sentence Prediction](/2021NTUML/Week8/next.JPG)
BERT訓練除了Masling之外，同時還會做另一個方法，叫`Next Sentence Prediction`，首先將輸入的資料庫取兩個Sequence，這兩個Sequence間會加入分隔符號，最前面會加入一個特別的符號，然後將整個Sequence輸入BERT，輸出會是與輸入相同長度的Sequence，但只看最前面特殊符號的輸出，將這個輸出經過Linear Model後做一個二元分類的問題，判斷輸入的兩個Sequence是否是相接的，是相接的話輸出Yes，反之輸出No。但後來研究發現`Next Sentence Prediction`對BERT後來要做的事沒有太大的幫助，後來還有另一個改進的做法叫` Sentence order prediction`，輸入兩個相連的句子讓BERT判斷是正接還是反接的，被用於BERT的進階版`ALBERT`

想了解更多可參考以下連結 : 
* RoBERTa: A Robustly Optimized BERT Pretraining Approach : [Link](https://arxiv.org/abs/1907.11692)
* ALBERT: A Lite BERT for Self-supervised Learning of Language Representations : [Link](https://arxiv.org/abs/1909.11942)

![BERT 1](/2021NTUML/Week8/bert-1.JPG)
因此訓練時，BERT要學兩個任務，一個是遮蔽一些輸入要BERT補回來，另一個是判斷兩個句子是否接在一起。感覺BERT只學會填補資料，但實際上他可以完成其他任務，這些任務叫`Downstream Tasks`，但要BERT做這些任務還是需要有一些標註的資料。將BERT微調去做其他任務叫做`Fine-tune`，而產生BERT的過程叫做`Pretrain`

