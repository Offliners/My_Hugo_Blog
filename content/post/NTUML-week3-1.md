---
title: "NTU - 機器學習 Week 3 - Tips for Training: Adaptive Learning Rate"
date: 2021-04-06T14:24:24+08:00
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

![cover](/2021NTUML/Week3/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video](https://www.youtube.com/watch?v=HYUXEeh3kwY)

Youtube Video Link (English): [Video](https://www.youtube.com/watch?v=8yf-tU7zm7w)

## Training Stuck equals Small Gradient?
![Training Stuck equals Small Gradient?](/2021NTUML/Week3/error-surface.JPG)
在訓練network時，通常會記錄loss，會發現loss越來越小，然後就卡住了不再下降，通常會覺得是遇到critical point，但真的是如此嗎?多數人可能不會確認說此時的gradient真的是否很小。上圖例子可以發現，loss不再下降，但gradient沒有很小。

![Training Stuck equals Small Gradient? -1](/2021NTUML/Week3/error-surface-1.JPG)
假設今天有個error surface，在參數b的變化非常小，參數w非常陡峭。如果learning rate設0.01時，會在山壁上震盪，無法滑進山谷。但learning rate設0.0000001時，雖然進入山谷，但仍然無法到終點，因為learning rate太小，步伐過小使參數要更新過多次也無法踏出多大的距離

會有這樣的問題在於learning rate是固定的，因此learning rate需要為每個參數客製化

## Different parameters needs different learning rate
![learning rate](/2021NTUML/Week3/lr-1.JPG)
大原則 : 在某個方向上，gradient很小非常平坦，會希望有較大的learning rate;gradient很大非常陡峭，會希望有較小的learning rate

為簡單起見，只用一個參數來講解。

將原本的learning rate除以一個參數，這個參數是depend on i，為每個參數客製化，接下來看說這個參數有什麼常見的計算方式

### Adagrad
![learning rate 2](/2021NTUML/Week3/lr-2.JPG)
首先是，過去所有gradient的均方根

![learning rate 3](/2021NTUML/Week3/lr-3.JPG)
這被使用在`Adagrad`的方法中

![learning rate 4](/2021NTUML/Week3/lr-4.JPG)
那這會有什麼問題呢?由於每個參數會需要的learning rate可能會隨時間改變，因此會希望同參數同方向，也能有動態調整的learning rate

### RMSProp
![learning rate 5](/2021NTUML/Week3/lr-5.JPG)
透過alpha來調整現在與過去的gradient的重要性，這個alpha也是一個超參數要自行設定。如果alpha設接近0，表示現在的gradient比較重要。反之設接近1，表示過去的gradient比較重要

![learning rate 6](/2021NTUML/Week3/lr-6.JPG)
透過此機制，可以視gradient來動態調整learning rate

### Adam
![learning rate 7](/2021NTUML/Week3/lr-7.JPG)
最常使用的，Adam是RMSProp結合Momentum

## Training with Adagrad
![Training with Adagrad](/2021NTUML/Week3/tr-lr.JPG)
之前沒有使用Adagrad來訓練，update參數10萬次只左一小步。但有了Adagrad來訓練，update參數10萬次較到達接近終點的地方，因為在平緩的地方會自動條大learning rate。那為什麼在終點時爆炸了呢?由於Adagrad是累積過去的gradient，因此在縱軸上每次累積很小的gradient，到一定程度時爆走，但沒關係，當衝出去時到大的gradient，會使sigma變大那update參數的步伐又會變小又回到峽谷，那如何解決這問題呢?

![Learning Rate Scheduling](/2021NTUML/Week3/lr-8.JPG)
隨著時間的增加，距離終點也越近，讓learning rate越來越小，使參數更新能慢下來，因此能解決剛剛的狀況

![Learning Rate Scheduling](/2021NTUML/Week3/warm-up.JPG)
除了decay的方法，還有warm-up，讓learning rate先變大後變小，變大變小的速率也是一種超參數。

![Paper](/2021NTUML/Week3/paper.JPG)
warm-up在訓練BERT常用到，此外在早期模型也常用到，如要有更多理解可以參考RAdam(https://arxiv.org/abs/1908.03265)這篇論文

## Summary of Optimization
![Summary of Optimization](/2021NTUML/Week3/summary.JPG)
有可能會有人問momentum與sigma都是考慮過去所有的gradient，那一個在分子一個在分母不會抵銷嗎?但momentum有考慮gradient的方向，但sigma是取平方相加，因此只考慮gradient的大小。因此不會互相抵銷

## To Learn More
* Optimization for Deep Learning(Chinese) [Video 1](https://youtu.be/4pUmZ8hXlHM) [Video 2](https://youtu.be/e03YKGHXnL8)
