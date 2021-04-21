---
title: "NTU - 機器學習 Week 3 - CNN"
date: 2021-04-08T21:12:19+08:00
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

![cover 2](/2021NTUML/Week3/cover-2.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video`](https://www.youtube.com/watch?v=OP5HcXJg2Aw)

Youtube Video Link (English): [`Video`](https://www.youtube.com/watch?v=I4eLIsPM9Yc)

## Image Classification
![Image Classification](/2021NTUML/Week3/image_classification.JPG)
通常會假設輸入圖片大小是固定的，如果圖片大小不一樣，會先Rescale。

### Neuron Version Story about Convolutional Layer 
![Input](/2021NTUML/Week3/input.JPG)
通常一張彩色圖片會是3維張量，分別為圖片的長、寬與Channel，這三個Channel為圖片的RGB。接著會把tensor給拉直，變成一維向量，那要如何拉直呢?將每個channel的數值排一排，就可以當成network的輸入，這個向量的每個數值，代表該顏色的強度。

![Input 2](/2021NTUML/Week3/input2.JPG)
將這個向量輸入到fully-connected network，如果輸入有100 * 100 * 3個feature，經過第一層假設有1000個神經元，那這一層會有30000000個weight。那這麼多參數會有什麼問題?雖然參數增加可以增加模型的彈性與能力，但也增加了訓練overfit的機率。考慮到影像的自身特性，其實不需要每個feature都有一個weight。那影像有什麼特性呢?

### Observation 1
![Input 3](/2021NTUML/Week3/input3.JPG)
對於影像辨識而言，希望神經元內能辨識的是圖片部分的pattern。比如圖片出現鳥嘴、鳥眼與鳥腳，因此能辨識出這是隻鳥

![Input 4](/2021NTUML/Week3/input4.JPG)
因此神經元不需要逐一對應到每一個特徵，神經元只需要對應到小區域的特徵群就好

![Simplification 1](/2021NTUML/Week3/simplification-1.JPG)
在CNN中，會定義一個區域叫`Receptive field`，這範圍內的特徵會被拉直輸入到一個神經元中，因此神經元只需要輸入3 * 3 * 3個特徵，相較之前的100 * 100 * 3少了許多

![Simplification 2](/2021NTUML/Week3/simplification-2.JPG)
那要怎麼決定Receptive field呢?這要自行決定。且Receptive field可以彼此重疊，甚至多個神經元去守備相同的Receptive field。

那Receptive field可以有不同大小嗎? `可以`

那Receptive field可以有只考慮一個channel嗎? `可以`

那Receptive field可以用長方形嗎? `可以`

![Simplification 3](/2021NTUML/Week3/simplification-3.JPG)
雖然Receptive field可以自行設定，那這邊要介紹經典的安排方式

會涵蓋所有的channel，因為深度會全涵蓋，所以只需要講高和寬，這個高和寬是`kernel size`，通常不會太大，常見的是(3, 3)

Receptive field會由一組多個神經元去守備，不會只有一個

Receptive field會移動到新的Receptive field，移動的長度是`stride`，通常不會太大(通常設1或2)，這樣才能重疊。假設Receptive field完全沒有重疊的話，如果有個pattern出現在兩個Receptive field的重疊處時，就會沒有神經元去偵測，就會遺失這個pattern。因此會希望Receptive field高度重疊

那如果Receptive field移動到超出範圍呢?如果邊緣不用Receptive field可能會遺失邊緣的pattern，所以通常會將超出邊緣的Receptive field做`padding`。補值方法很多，可以填0、填整張圖的平均或者填邊緣值等

以上是第一個簡化fully connected network的方式

### Observation 2
![Simplification 4](/2021NTUML/Week3/simplification-4.JPG)
那還能怎麼簡化呢?由於多數神經元是做相同的一件事(如偵測鳥嘴)，只是守備的Receptive field不同。既然如此，真的需要每個Receptive field去放一個神經元嗎?這樣參數可能需要太多

![Simplification 5](/2021NTUML/Week3/simplification-5.JPG)
為了解決上述問題，需要神經元能共享參數

![Simplification 6](/2021NTUML/Week3/simplification-6.JPG)
由於守備的Receptive field不同使輸入不同，因此即使weight相同不會導致輸出相同。所以不會讓兩個守備相同Receptive field的神經元共享參數，因為輸入相同，weight也相同，那這兩個神經元的輸出也會相同

![Simplification 7](/2021NTUML/Week3/simplification-7.JPG)
在影像辨識中，會使每組守備不同Receptive field的神經元，使用相同的一組參數，而這參數叫`Filter`

### Benefit of Convolutional Layer
![## Benefit of Convolutional Layer](/2021NTUML/Week3/benefit.JPG)
Fully connected layer會看整張圖片或者自訂的範圍，Receptive field只能看小範圍，因此network的彈性更小，再加入Parameter Sharing使某些神經元只能用同一組參數不能訓練出新的參數。
將Receptive field加上Parameter Sharing就是Convolutional Layer，由於限制大，因此model bias也大。

Fully connected layer由於模型彈性大，所以可以做各項任務，但容易overfitting使得沒辦法將特定任務做好。而Convolutional Layer是特定為影像而設計的，因此在影像領域上的任務能做好，但影像外的任務可能就沒辦法做好

## Filter Version Story about Convolutional Layer
![Convolutional Layer](/2021NTUML/Week3/cnn.JPG)
channel視輸入圖片而定，如果是RGB，那channel為3。如果是黑白，那channel為1。每個filter會到3 * 3 * channel中抓取pattern

![Convolutional Layer 1](/2021NTUML/Week3/cnn-1.JPG)
假設現在輸入是6 * 6的黑白圖片，並且假設filter中的tensor是已知的(實際上是未知的，要透過梯度下降法找出)

![Convolutional Layer 2](/2021NTUML/Week3/cnn-2.JPG)
將filter 1與第一個區域做內積得3，接著向右移動stride步，再做內積，以此類推。filter 1偵測pattern為找內積結果最大的區域，在這例子中可以得知左上角與左下角有出現filter 1要尋找的pattern

![Convolutional Layer 3](/2021NTUML/Week3/cnn-3.JPG)
filter 1也是相同的做法來尋找特定的pattern。經過filter得到的數字叫`feature map`

![Convolutional Layer 4](/2021NTUML/Week3/cnn-4.JPG)
如果有64個filter，那就可以得到類似圖片的feature map，維度從3變成64。因此下一層Convolutional Layer的filter要抓取pattern的區域大小就會變成3 * 3 * 64

![Convolutional Layer 5](/2021NTUML/Week3/cnn-5.JPG)
Q : 如果feature只有3 * 3，會不會使network沒辦法看大範圍的pattern?

A : 不會。就第一層Convolutional Layer的feature map來看，雖然第二層filter內積區域只有3 * 3，但這範圍在原來的影像上其實是考慮5 * 5範圍。所以network夠深，看的範圍越大，可以偵測到較大的pattern

## Comparison of two stories
![Comparison of two stories 1](/2021NTUML/Week3/cmp.JPG)
在實際CNN運算是有bias的

![Comparison of two stories 2](/2021NTUML/Week3/cmp-2.JPG)
第一個故事的share weight就是第二個版本filter掃過區域，也就是`Convolution`

![Comparison of two stories 3](/2021NTUML/Week3/cmp-3.JPG)

## Pooling - MaxPooling
![Pooling - MaxPooling](/2021NTUML/Week3/pooling.JPG)
由於Pooling中沒有參數，所以不是Layer，比較像是activation function。舉MaxPooling為例，將feature map每個channel的數字分組，並選組中最大的值，一組要多大也是自行決定

![Pooling - MaxPooling 1](/2021NTUML/Week3/pooling-1.JPG)
實作上，Convolution會與Pooling交替使用，可能幾層Convolution後接Pooling。由於Pooling就是一種subsampling，對於重要的小特徵的表現會差一些。近年來影像設計的模型會捨棄Pooling，反而做Full Convolution。Pooling存在最主要的理由是減少運算量，由於運算能力提升，因此最近架構就不做Pooling，而是全Convolution

## The whole CNN
![The whole CNN](/2021NTUML/Week3/cnn-6.JPG)
經典影像辨識架構 : 將Convolution Layer後接Pooling，重複幾次後Flatten後接fully connected network，最後經過softmax來辨識圖片內容

## Application : Playing Go
![Application : Playing Go](/2021NTUML/Week3/AlphaGo.JPG)
下圍棋其是一種分類問題，輸入棋盤黑白子位置，輸出下一步可以落子位置。這問題可以使用fully connected network，但CNN效果更好。在AlphaGo的論文中，棋子位置是用48個channel來表示

### Why CNN for Go Playing?
![Why CNN for Go Playing?](/2021NTUML/Week3/why.JPG)
那為何CNN能運用在圍棋上?首先因為圍棋跟影像相同，重要Pattern只需看小區域即可(Observation 1)，且同樣的Pattern會出現不同位置(Observation 2)。因此影像與圍棋有很多共同之處。但影像可以使用Pooling或subsampling，但圍棋不行，因為可能會改變棋局

![Why CNN for Go Playing? 1](/2021NTUML/Week3/why-1.JPG)
AlphaGo將棋盤視為19 * 19 * 48的圖片，且使用zero-padding填補邊界，有192個5 * 5的filter，且stride為1，並使用ReLU作為activation function，最後使用softmax來選擇下一步的落子。重點是，整個網路架構沒有使用Pooling

## More Application
![More Application](/2021NTUML/Week3/more.JPG)
若想使用CNN在影像其他領域的話(如語音、文字處理上)，需仔細看文獻，在其他領域有不同的Receptive field與參數共享，而是考慮其他領域的特性來設計

CNN沒辦法處理影像縮放與旋轉的問題，所以需要`Data Augmentation`，或者使用`Spatial Transformer Layer`來處理

## To Learn More
Spatial Transformer Layer : [Link](https://youtu.be/SoCywZ1hZak)