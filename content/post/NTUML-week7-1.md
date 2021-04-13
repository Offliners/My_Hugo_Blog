---
title: "NTU - 機器學習 Week 7 - Transformer"
date: 2021-04-13T20:43:16+08:00
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

![cover 1](/2021NTUML/Week7/cover-1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video 1](https://www.youtube.com/watch?v=n9TlOhRjYoc) [Video 2](https://youtu.be/N6aRv06iv2g)

Youtube Video Link (English): [Video 1](https://www.youtube.com/watch?v=zmOuJkH9l9M) [Video 2]

## Applications of Sequence-to-sequence (Seq2seq)
![Sequence-to-sequence](/2021NTUML/Week7/seq2seq.JPG)
當模型輸入是sequence時，輸出可以是與輸入相同長度的sequence，也可能是未知長度的sequence，由機器去決定。輸出是未知長度sequence的例子如
* 語音辨識 Speech Recognition
* 機器翻譯 Machine Translation
* 語音翻譯 Speech Translation

這時可能有人問為何要做語音翻譯?直接語音辨識再機器翻譯不就好了?因為世界上有眾多語言，有些語言甚至連文字都沒有，沒有文字的語言難以做語音辨識

若想了解台語語音辨識可點以下連結:
To learn more : [Link](https://sites.google.com/speech.ntut.edu.tw/fsw/home/challenge-2020)

輸入聲音，輸出辨識後的文字是`語音辨識 Speech Recognition`。反過來，輸入文字，輸出聲音訊號，就是`語音合成 Text-to-Speech Synthesis`

### Seq2seq for Chatbot
![Seq2seq for Chatbot](/2021NTUML/Week7/chatbot.JPG)
文字方面，也可透過使用Seq2seq的模型來訓練，例如可以訓練聊天機器人，輸入輸出都是文字，訓練資料是一堆對話的內容

### Most Natural Language Processing applications
![Most Natural Language Processing applications](/2021NTUML/Week7/NLP.JPG)
很多NLP的問題可以看成是QA的問題，多數QA的問題又可以透過Seq2seq解決。輸入問題與文章，輸出就是問題的答案

想了解更多NLP的處理，可參考以下連結
To learn more : 
* The Natural Language Decathlon: Multitask Learning as Question Answering : [Link](https://arxiv.org/abs/1806.08730)
* LAMOL: LAnguage MOdeling for Lifelong Language Learning : [Link](https://arxiv.org/abs/1909.03329)
* DEEP LEARNING FOR HUMAN LANGUAGE PROCESSING 2020 SPRING : [Link](https://speech.ee.ntu.edu.tw/~hylee/dlhlp/2020-spring.html)

### Seq2seq for Syntactic Parsing
![Seq2seq for Syntactic Parsing](/2021NTUML/Week7/syntacticparsing-1.JPG)
Seq2seq也可用於文法剖析，輸入是一段文字，但輸出就不是sequence，而是樹狀結構用來分析各文字的組成

![Seq2seq for Syntactic Parsing](/2021NTUML/Week7/syntacticparsing-2.JPG)
不過樹狀結構也可以看成是一段sequence，這樣就可以套用Seq2seq來處理

To learn more : 
* Grammar as a Foreign Language : [Link](https://arxiv.org/abs/1412.7449)

### Seq2seq for Multi-label Classification
![Seq2seq for Multi-label Classification](/2021NTUML/Week7/multilabel.JPG)
首先要區分`Multi-label Classification` 與`Multi-class Classification`。Multi-class Classification是多個class由機器去選擇一個，Multi-label Classification是指同一個東西但屬於可能有多個class。那要怎麼處理Multi-label Classification的問題呢?可以透過Seq2seq，比如輸入文章，輸出是他所屬於的class

To learn more : 
* Order-free Learning Alleviating Exposure Bias in Multi-label Classification : [Link](https://arxiv.org/abs/1909.03434)
* Order-Free RNN with Visual Attention for Multi-Label Classification : [Link](https://arxiv.org/abs/1707.05495)

### Seq2seq for Object Detection
![Seq2seq for Object Detection](/2021NTUML/Week7/object.JPG)
甚至連物件偵測也能透過Seq2seq來解決

To learn more : 
* End-to-End Object Detection with Transformers : [Link](https://arxiv.org/abs/2005.12872)

## Seq2seq
![Seq2seq](/2021NTUML/Week7/seq2seq-1.JPG)
一般的Seq2seq會由encoder與decoder組成，上圖是Seq2seq早期的架構

To learn more : 
* Sequence to Sequence Learning with Neural Networks : [Link](https://arxiv.org/abs/1409.3215)

![Transformer](/2021NTUML/Week7/transformer.JPG)
現今主流架構會是`Transformer`

## Encoder
![Encoder](/2021NTUML/Week7/encoder.JPG)
Encoder在做的事情是輸入一排向量，輸出也是一排向量。Transformer中的encoder就是self-attention，透過下圖來簡單介紹

![Transformer Emcoder 1](/2021NTUML/Week7/transformer-1.JPG)
上圖每個block都是輸入一排向量，輸出一排向量。而block中會先對輸入做self-attention，接著送入fully connected network中，最後就是輸出的向量

![Transformer Emcoder 2](/2021NTUML/Week7/transformer-2.JPG)
而實際的Transformer中，會把self-attention輸出的向量加上原本的輸入，這樣的行為叫做`Residual`。接著會把residual後的向量做`Layer Normalization`，不同於Batch Normalization的是Batch Normalization是對同一個維度不同的feature做計算，但Layer Normalization不同的維度同一個的feature做計算。做完Layer Normalization得到的輸出就會是fully connected network的輸入，這裡也會套用residual與Layer Normalization的架構

![Transformer Emcoder 3](/2021NTUML/Week7/transformer-3.JPG)
上述介紹的簡易模型也就是Transformer encoder中block處理的方式，而這複雜的block再之後會介紹的BERT也會用到

若想了解更多為何要這樣encoder，可參考以下連結
To learn more : 
* On Layer Normalization in the Transformer Architecture : [Link](https://arxiv.org/abs/2002.04745)
* PowerNorm: Rethinking Batch Normalization in Transformers : [Link](https://arxiv.org/abs/2003.07845)
