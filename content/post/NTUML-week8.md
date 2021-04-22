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

## GLUE
![GLUE](/2021NTUML/Week8/glue.JPG)
要測試一個模型Self-supervised Learning的能力，會測試多個任務，在多個任務中的正確率取平均值，這樣的任務集最有名的是`General Language Understanding Evaluation (GLUE)`，分別微調在9個任務上

![BERT scores](/2021NTUML/Week8/bertscore.JPG)
以上是BERT相關的模型在GLUE的得分，可以發現最近較強的模型在這些資料集中的平均得分超越人類的表現

## How to use BERT
### Case 1
![How to use BERT 1](/2021NTUML/Week8/case-1.JPG)
Case 1是輸入Sequence輸出一個類別，舉`Sentiment analysis`為例，將輸入的句子前面放置CLS這個特殊的token，接著得到一排向量的輸出，但只看CLS對應的輸出，接著經過Linear模型與softmax來判斷是正面還是負面。但仍需要提供BERT一些標註的資料，且Linear模型的參數是隨機初始化，但BERT是透過Fine-tune出來的，是已經被預訓練的模型，因此BERT的參數不是隨機初始化的

![Pre-train v.s. Random Initialization](/2021NTUML/Week8/vs.JPG)
上圖比較可以發現有Fine-tune的模型(實線)loss下降較快且可以到比較低的位置，但沒有Fine-tune的模型(虛線)，也就是透過Scratch的方法，loss下降較慢且最終得到的loss也較大

### Case 2
![How to use BERT 2](/2021NTUML/Week8/case-2.JPG)
Case 2是輸入Sequence輸出是與輸入相同長度的Sequence，舉`POS Tagging`為例，與Case 1不同的是輸出是看CLS以外的輸出，由於要輸出相同長度的Sequence，所以不會看CLS的輸出，接著輸入至Linear模型再經過Softmax來判斷說這是什麼詞性

### Case 3
![How to use BERT 3](/2021NTUML/Week8/case-3.JPG)
Case 3是輸入兩個Sequence輸出一個類別，舉`Natural Language Inferencee (NLI)`為例，輸入的兩個句子，一個是前提另一個是假設，機器要用前提與假設判斷是否矛盾

![How to use BERT 3-1](/2021NTUML/Week8/case-3-1.JPG)
BERT會輸入兩個句子組成的Sequence，並從CLS對應的輸出來判斷是否矛盾

### Case 4
![How to use BERT 4](/2021NTUML/Week8/case-4.JPG)
接下來要介紹的問題是`Extraction-based Question Answering (QA)`，這是一種問答系統，且答案一定會出現在文章中。首先會將文章D與問題Q輸入到模型中，且會輸出兩個正整數s、e，這兩個正整數代表文章的第s個字到第e個字，串起來就是機器會輸出的答案

![How to use BERT 4-1](/2021NTUML/Week8/case-4-1.JPG)
與Case 3相同，輸入為文章與問題組成的Sequence，接著BERT會輸出一串Sequence，然後將文章對應輸出的Sequence對橘色的向量做內積，再做softmax，得到數值最高的位置為s

![How to use BERT 4-2](/2021NTUML/Week8/case-4-2.JPG)
再來對文章對應輸出的Sequence對藍色的向量做內積，再做softmax，得到數值最高的位置為e，這兩個顏色的向量是機器要訓練的，且向量是隨機初始化的

### Training BERT is challenging
![Training BERT is challenging](/2021NTUML/Week8/challenge.JPG)
要訓練BERT非常難，因為參數量過多因此十分耗時

## Pre-training a seq2seq model
![Pre-training a seq2seq model](/2021NTUML/Week8/seq2seq.JPG)
由於BERT是將Transformer的encoder先預訓練好，那如果要解seq2seq的問題，也可以預訓練decoder? 可以，會將輸入至encoder的資料增加一些擾動，然後預期deocder的輸出與沒有擾動的資料是相同的

![Pre-training a seq2seq model 1](/2021NTUML/Week8/mass.JPG)
那要怎麼增加擾動呢? 有許多種方法，如使用特殊的符號遮蔽、直接刪除、弄亂順序、順序旋轉等

![T5](/2021NTUML/Week8/T5.JPG)
Google已經有提供T5這個預訓練的模型，將多種擾動的方式用於其中

## Why does BERT work?