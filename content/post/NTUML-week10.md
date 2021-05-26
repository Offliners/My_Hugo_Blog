---
title: "NTU Machine Learning Week 10 - Auto-Encoder"
date: 2021-05-01T18:21:00+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 10 - Auto-Encoder

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week10/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/3oHlf8-J3Nc) [`Video 2`](https://youtu.be/JZvEzb5PV3U)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=E7wlA85RxcI) [`Video 2`](https://www.youtube.com/watch?v=PsBHWq9KKqk)

## Self-supervised Learning Framework
![Self-supervised Learning Framework](/2021NTUML/Week10/self.JPG)
將一些沒有標註的資料給模型訓練叫`Self-supervised Learning`或者`Pre-train`，訓練完後的模型做些微調整後可以應用在其他任務上。而`Auto-encoder`也是不需要標註資料的模型，因此也是一種Self-supervised Learning

## Auto-encoder
![Auto-encoder](/2021NTUML/Week10/autoencoder.JPG)
在Auto-encoder中有兩個network，一個是Encoder，另一個是Decoder。假設有一堆沒有標註的圖片，會輸入到Encoder中輸出成向量，這個向量再輸入至Decoder，輸出成一張圖片，Decoder運作類似GAN中的Generator。訓練目標是希望Decoder輸出的圖片與輸入Encoder的圖片越接近越好，而這件事叫做`Reconstruction`，行為類似Cycle GAN。中間的向量有人稱為`Embedding`、`Representation`或`Code`。通常Encoder的輸入會是高維度的向量，而Encoder的輸出會是低維度的向量，所以Encoder做的事就是`Dimension Reduction`

若想了解更多Dimension Reduction技術可以參考以下連結 : 
* Unsupervised Learning - Linear Methods : [Video](https://youtu.be/iwh5o_M4BNU)
* Unsupervised Learning - Neighbor Embedding : [Video](https://youtu.be/GBUEjkpoxXc)

## Why Auto-encoder?
![Why Auto-encoder?](/2021NTUML/Week10/why.JPG)
Auto-encoder中的Encoder能將複雜的圖片變成較簡單向量來表示

![Why Auto-encoder?](/2021NTUML/Week10/idea.JPG)
Auto-encoder的想法已經持續許久，早在2006就已經有了

## De-noising Auto-encoder
![De-noising Auto-encoder](/2021NTUML/Week10/denoise.JPG)
De-noising Auto-encoder會將輸入圖片加入一些雜訊，試圖讓Auto-encoder還原沒有雜訊的圖片，讓模型學會濾除雜訊的能力

![BERT](/2021NTUML/Week10/bert.JPG)
而BERT做的事情就像是De-noising Auto-encoder

## Feature Disentangle
![Feature Disentangle](/2021NTUML/Week10/disentangle.JPG)
Auto-encoder在Encoder中會將複雜的高維向量變成簡單的code再透過Decoder變回複雜的高維向量，而這些code內有重要的資訊，但哪些維度代表什麼資訊並不知道

![Feature Disentangle 1](/2021NTUML/Week10/disentangle-1.JPG)
因此有了`Feature Disentangle`的議題，在訓練Auto-encoder時Encoder輸出的向量，哪些維度代表哪些資訊

若想了解Feature Disentangle是如何實現可以參考以下論文 : 
* One-shot Voice Conversion by Separating Speaker and Content Representations with Instance Normalization : [Link](https://arxiv.org/abs/1904.05742)
* Multi-target Voice Conversion without Parallel Data by Adversarially Learning Disentangled Audio Representations : [Link](https://arxiv.org/abs/1804.02812)
* AUTOVC: Zero-Shot Voice Style Transfer with Only Autoencoder Loss : [Link](https://arxiv.org/abs/1905.05879)

## Application: Voice Conversion
![Application: Voice Conversion](/2021NTUML/Week10/voice.JPG)
在過去要做語者轉換需要兩種不同音色的人講相同的話來進行Supervised learning，但這非常不實際。使用Feature Disentangle的技術，可以做到透過訓練不同的話來進行語者轉換

![Application: Voice Conversion 1](/2021NTUML/Week10/voice-1.JPG)
透過Feature Disentangle，了解哪些維度是內容，哪些是聲音特徵，將內容結合別人的聲音特徵來進行語者轉換

## Discrete Latent Representation
![Discrete Latent Representation](/2021NTUML/Week10/discrete.JPG)
之前Auto-encoder中的code是一個任一值的向量，那可以限制說輸出是一個Bineay的向量，每一維代表是否有某些特定的特徵，也可以是one-hot向量，讓模型訓練Unsupervised的分類，例如輸入手寫數字，輸出10維的向量，訓練模型分類沒有標籤的手寫數字

![VQVAE](/2021NTUML/Week10/VQVAE.JPG)
這樣的做法最有名的模型是`VQVAE`，輸入一張圖片讓Encoder輸出一個向量，去跟Codebook中的向量計算相似度，Codebook中的向量也是透過模型學習出來的，將相似度最高的向量輸入Decoder來產生圖片

![Discrete Latent Representation 1](/2021NTUML/Week10/discrete-1.JPG)
Code不一定要是向量，也可以是一段文字，由於輸入是文字所以需要用Seq2seq的模型，這種`Seq2seq2seq Auto-encoder`會將長sequence變成短的再變成長的，預期上之間的Code可能會是文章的摘要，因為這是文章的重要內容，但實際上會發現不是，因為Encoder與Decoder會發明之間的暗號

![Discrete Latent Representation 2](/2021NTUML/Week10/discrete-2.JPG)
因此需要再透過GAN中的Discriminator來判斷是否是人能讀懂的句子。所以Encoder需要能輸出一段句子，這句子要能騙過Discriminator，又能使Deocder還原原來的文章，而這樣的概念就是CycleGAN

![Tree as Embedding](/2021NTUML/Week10/tree.JPG)
甚至連樹狀結構都能當成Code，若想了解更多可以參考以下論文 : 
* StructVAE: Tree-structured Latent Variable Models for Semi-supervised Semantic Parsing : [Link](https://arxiv.org/abs/1806.07832)
* Unsupervised Recurrent Neural Network Grammars : [Link](https://arxiv.org/abs/1904.03746)

## More Applications 
### Generator
![Generator](/2021NTUML/Week10/generator.JPG)
由於Decoder在做的事與GAN中的Generator相同，輸入像量產生圖片，這就是VAE的作法，訓練Auto-encoder，再將Decoder當成Generator使用

### Compression
![Compression](/2021NTUML/Week10/compression.JPG)
由於Auto-encoder中的Encoder會將高維向量變成低維向量，因此可以做壓縮，但這是一種lossy的壓縮，表示會失真

### Anomaly Detection
![Anomaly Detection](/2021NTUML/Week10/anomaly.JPG)
Auto-encoder可以做異常檢測，準備一筆訓練資料，當輸入一個測試資料，如果類似訓練資料則是正常，反之則異常

以下為常見異常檢測技術的應用與相關連結 : 
* 詐欺偵測
    - https://www.kaggle.com/ntnu-testimon/paysim1/home
    - https://www.kaggle.com/mlg-ulb/creditcardfraud/home
* 網路的侵入偵測
    - http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html
* 癌症檢測
    - https://www.kaggle.com/uciml/breast-cancer-wisconsin-data/home

那異常檢測是否可以當成二元分類的問題?由於通常資料中正常資料數量遠大於異常資料，因此常視為one-class，只有一個類別那要無法訓練二元分類的模型，那Auto-encoder要如何訓練呢?

![Anomaly Detection 1](/2021NTUML/Week10/anomaly-1.JPG)
假設要識別是否為真人人臉，輸入真人人臉給Auto-encoder來訓練，測試時輸入真人人臉因為訓練時可能看過類似的，所以Decoder能順利還原

![Anomaly Detection 2](/2021NTUML/Week10/anomaly-2.JPG)
但如果輸入二次元人臉，因為訓練時完全沒看過，所以Decoder還原較差，因此能判斷是異常

若想了解更多Anomaly Detection的技術，可以參考以下連結 : 
* Anomaly Detection : [Video 1](https://youtu.be/gDp2LXGnVLQ) [Video 2](https://youtu.be/cYrNjLxkoXs) [Video 3](https://youtu.be/ueDlm2FkCnw) [Video 4](https://youtu.be/XwkHOUPbc0Q) [Video 5](https://youtu.be/Fh1xFBktRLQ) [Video 6](https://youtu.be/LmFWzmn2rFY) [Video 7](https://youtu.be/6W8FqUGYyDo)