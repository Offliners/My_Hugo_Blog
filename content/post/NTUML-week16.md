---
title: "NTU Machine Learning Week 16 - Reinforcement Learning"
date: 2021-06-11T17:14:52+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 16 - Reinforcement Learning

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week16/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/XWukX-ayIrs) [`Video 2`](https://youtu.be/US8DFaAZcp4) [`Video 3`](https://youtu.be/kk6DqWreLeU) [`Video 4`](https://youtu.be/73YyF1gmIus) [`Video 5`](https://youtu.be/75rZwxKBAf0)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=0oucZNfBXlI) [`Video 2`](https://youtu.be/jbN0oYLtXps) [`Video 3`] [`Video 4`] [`Video 5`]

## What is RL ?
![ML](/2021NTUML/Week16/ML.JPG)
機器學習最主要在做的事情就是找一個Function，Reinforcement Learning也是機器學習的一種，所以它也是在找一個Function。在RL中會有個Actor與Environment，首先Environment會給Actor一個Observation，也就是Actor的輸入，接著Actor會有輸出Action去影響Environment來產生新的Observation，互動期間Environment會給Actor一些Reward來告訴Actor這樣的Action是否是好的。所以Actor其實就是我們要找的Function，它要能讓Actor得到Reward的總和是最大的

## Example: Playing Video Game
![Game](/2021NTUML/Week16/game.JPG)
接下來使用Space invader來舉例，遊戲玩法是開火擊退所有外星人，而玩家有護盾可以抵擋外星人的攻擊(玩家的攻擊也會破壞護盾)，擊退外星人可以獲得分數。遊戲終止有兩個情況，一個是退所有外星人，另一個情況是玩家被外星人擊毀

![Game 1](/2021NTUML/Week16/game-1.JPG)
當要訓練RL來玩這遊戲的話，Actor就是玩家，Environment是遊戲本身，Observation是遊戲畫面，Action是輸出，有三種可能的行為分別為向左向右還有開火。如果Action選擇向右的話，reward是0因為沒有擊退外星人

![Game 2](/2021NTUML/Week16/game-2.JPG)
當遊戲畫面變化時，表示有新的Observation，這時就會有新的Action，如果選擇開火並擊退外星人的話，就會得到reward等於5。因此學習的目標是，讓Actor能得到的reward總和是最大的

## Example: Learning to play Go
![Alpha Go](/2021NTUML/Week16/go.JPG)
若是Alpha Go的話，Environment是人類對手，Observation就是棋盤，Action就是落子的位置，透過對手的落子來產生新的Observation，然後持續下去

![Alpha Go 1](/2021NTUML/Week16/go-1.JPG)
在下圍棋中，任何行為都沒辦法得到reward，因此會定義贏了才能得到reward為1，輸了得到-1分

## How to train RL ?
![train](/2021NTUML/Week16/train.JPG)
那要怎麼訓練RL呢?與先前的相同，第一步是有未知數的Functin，這些未知數是要被找出來的，第二步是定義loss function，第三步是最小化loss

![Step 1](/2021NTUML/Week16/step1.JPG)
首先是第一步，會使用Network，輸入是遊戲畫面，輸出是每個行為的分數，這件事情其實就是分類。Network架構可以自行設計，若輸入是遊戲畫面可能會使用CNN，若想看遊戲開始到現在發生的情況就有可能使用RNN或者Transformer。最後機器會採取什麼Action取決於輸出的分數，通常會轉換成機率並隨機Sample，那為什麼不直接選擇機率最高的呢?多數RL的應用中，會選擇隨機Sample，這樣子即使看到相同畫面也有可能採取不同行為，來增加隨機性

![Step 2](/2021NTUML/Week16/step2.JPG)
第二步要定義Loss，在Environment與Actor的互動中會得到許多reward直到遊戲結束，這些reward總合起來叫Total reward或者Return，這是我們要去最大的目標，加上負號就是loss，也就是要最小化

![Step 3](/2021NTUML/Week16/step3.JPG)
最後是第三步，Environment會產生Observation叫s1，再來將s1輸入給Actor產生Action叫a1，持續下去到遊戲終止，這些s與a形成的Sequence叫做`Trajectory`。得到的Reward能當成是一個Function，有些遊戲只看Action就能決定，但通常會看當下的Observation，像Space invader需要觀察Observation是否擊退敵人才能判斷是否得到reward。但這不是一般的Optimization問題，首先是Actor的輸出是有隨機性，而且Environment是個黑盒子，只知道輸入後會產生回應，但對應關係我們不知道。因此在RL中解決Optimization問題使用的方法會與之前不同

## Policy Gradient