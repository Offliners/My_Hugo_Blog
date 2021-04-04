---
title: "NTU - 機器學習 Week 2 - Critical Point: small gradient"
date: 2021-04-05T00:38:19+08:00
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

![cover1](/2021NTUML/Week2/cover2.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [Video](https://www.youtube.com/watch?v=QW6uINn7uGk)

Youtube Video Link (English): [Video](https://youtu.be/yz7QS1I6omw)

## Why Optimization fails?
![Fail](/2021NTUML/Week2/fails.JPG)
有時會發現隨著參數不斷update，training的loss不再下降(藍線)，但對這loss仍不滿意。跟其他如linear model以較發現deep network沒有更好，顯然optimization有問題。有時還會發現一開始model就train不起來，不管怎麼update參數loss仍不降(紅線)。那這是發生什麼事呢?

過去的猜想是，由於模型訓練到一個地方，這地方參數對loss的微分為0，梯度下降法就沒辦法再update參數，所以loss不再下降。

說到Gradient為0，有兩種常見的可能:
1. Local Minima 區域最小值
2. Saddle Point 鞍點
3. Local Maxima 區域最大值

這些會使Gradient為0的點統稱`Critical point`，那如果今天Gradient很接近0，那是卡在哪一個呢?等等會解說如何判斷

那又為何需要分辨卡在哪裡?如果卡在local minima那就沒路可走，但如果是saddle point那旁邊還有路可以走，可以讓loss更低。因為update參數是往低處走，所以不太會卡在local maxima

## Math Method
要判斷是local minima還是saddle point呢?首先要知道loss function的形狀，由於模型通常過於複雜，使得我們無法得知loss function的全貌，但可以給定某一組參數來分析在這組參數下loss function附近的樣貌

### Tayler Series Approximation
![Tayler Series Approximation 1](/2021NTUML/Week2/tayler1.JPG)
透過泰勒級數近似法可以將loss function在這組參數下分成以上三個的組合

![Tayler Series Approximation 2](/2021NTUML/Week2/tayler2.JPG)
如果今天卡在critical point，表示gradient為0，那綠色那項也為0，因此可以根據紅色那項判斷這組參數附近的error surface是長什麼樣子，進而判斷卡在哪一種critical point

![Hessian](/2021NTUML/Week2/hessian.JPG)
為方便表示，將
$$ v = (\theta - \theta^{'})$$
1. 紅色項帶任何值皆大於0，表示這組參數是附近的最低點，因此是`local minima`

2. 紅色項帶任何值皆小於0，表示這組參數是附近的最高點，因此是`local maxima`

3. 紅色項帶任何值有時大於0有時小於0，表示這組參數在附近有時高有時低，因此是`saddle point`

但不可能帶入任何v值去確認，因此透過線性代數中，判斷H是正定矩陣、負定矩陣還是都不是，透過H矩陣的特徵值即可判斷

### Example
![Example 1](/2021NTUML/Week2/example1.JPG)
透過窮舉法，將所以loss function值求出來可以得到上圖的error surface，得知(0, 0)為saddle point

![Example 2](/2021NTUML/Week2/example2.JPG)
透過求H的特徵值發現有正與有負，因此(0, 0)確實為saddle point

## How to update weight at saddle point?
![Update 1](/2021NTUML/Week2/update1.JPG)
因此只要沿著特徵向量的方向，就可以讓loss變得更低

![Update 2](/2021NTUML/Week2/update2.JPG)
由上圖可知沿著(1, 1)方向可以找到比saddle point的loss更低的地方。實際上不會使用這方法來逃離saddle point，因為二次微分運算量大，更不用說求出特徵值與特徵向量

## Saddle Point v.s. Local Minima
![Saddle Point v.s. Local Minima 1](/2021NTUML/Week2/vs1.JPG)
用一維來看，感覺處處都是local minima，但在二維空間來看是saddle point。由於現在模型參數動輒千百萬，維度非常高，那會不會現在看到的local minima在更高維度上來看其實是saddle point，那維度越高是不是可以走的路越多?

![Saddle Point v.s. Local Minima 2](/2021NTUML/Week2/vs2.JPG)
透過實驗也支持此假說，上圖每個藍點是模型訓練完的結果，可以發現沒有任何一個結果真的卡在local minima(minimum ratio = 1或接近1)，因此local minima沒那常見，多數是卡在saddle point

