---
title: "NTU - 機器學習 Week 5 - Self-Attention"
date: 2021-04-10T23:06:41+08:00
draft: false
toc: true
comment: true

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover 1](/2021NTUML/Week5/cover-1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://www.youtube.com/watch?v=hYdO9CscNes) [`Vedio 2`](https://www.youtube.com/watch?v=gmsMY5kc-zw)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=0djMUi2-uV4) [`Vedio 2`](https://www.youtube.com/watch?v=zeCDPYZli0k)

## Sophisticated Input
![Sophisticated Input](/2021NTUML/Week5/input.JPG)
以往的輸入(預測Youtube觀看人數、影像處理等)都可以看成是一個向量，那如果輸入變成一排向量呢?
或者輸入許多大小不一的向量呢?

![Vector Set as Input 1](/2021NTUML/Week5/vectorset.JPG)
那有什麼輸入是Sequence且長度會改變的呢?如文字處理，輸入是多個句子，且句子長度不同。那要怎麼把詞彙變成像量呢?最簡單的做法是`one-hot encoding`，假如有個很長的向量，向量長度為世界上存在詞彙的數目，當出現某個詞彙，那該詞彙所在的index就會為1。除了需要非常長的向量之外會存在一個嚴重的問題，那就是詞彙之間沒有關係，即使詞彙都是動物的種類，但彼此沒有關係，因為這種表示法沒有語意的資訊。因此有`word emdeding`，想得知更多可觀看以下連結的影片

* To Learn More : Unsupervised Learning - Word Embedding [Video](https://youtu.be/X7PH3NuYW0Q)

![Vector Set as Input 2](/2021NTUML/Week5/vectorset-1.JPG)
還有聲音訊號也可當成長度不一的向量組，可以把聲音訊號取一個範圍，將這裡面的資訊描述成向量，也就是一個`Frame`，將一段訊號描述成向量有多種作法。為了描述整個整個向量會移動這個範圍，再將這裡面描述成新的向量，因此就能組成向量組

![Vector Set as Input 3](/2021NTUML/Week5/vectorset-2.JPG)
還有Graph也能看成是向量組，每個節點可以描述成向量，這些向量也能組成向量組來描述Graph

![Vector Set as Input 4](/2021NTUML/Week5/vectorset-3.JPG)
既然Graph能描述成向量組，那分子也可以，每個分子當成一個Graph，而每種元素能用one-hot encoding來描述成向量

## What is the output?
![What is the output? 1](/2021NTUML/Week5/output-1.JPG)
既然輸入是向量，那輸出是麼呢?首先輸入與輸出的向量可以對應到一個Label，如果Label對應到數值，那就是Regression問題;如果對應到class，那就是Classification問題，這種形式輸入輸出的長度是相同的。有以下應用:
* POS Tagging 詞性標註
* 語音處理
* Social Network Porblem

![What is the output? 2](/2021NTUML/Week5/output-2.JPG)
那還有輸入向量組，輸出只是一個Label。有以下應用:
* Sentiment Analysis 情緒分析
* 語者辨認
* 分子分析

![What is the output? 3](/2021NTUML/Week5/output-3.JPG)
最後一種可能的輸出是不知道要輸出多少Label，機器要自己決定，這種任務叫做`Sequence to Sequence`

本次課程將會著重在第一種類型

## Sequence Labeling
![Sequence Labeling](/2021NTUML/Week5/seqlabel.JPG)
這種輸入輸出相同長度的問題，叫做`Sequence Labeling`。那要怎麼解呢?可以使用Fully connected，將sequence的每個向量輸入到一個Fully connected network來進行輸出。但存在非常大的瑕疵，假設做詞性標記，如上圖舉例的例子，第一個saw是動詞，第二個是名詞，但機器不會知道。因此需要讓機器考慮到前後文的資訊，也就是將前後幾個向量也輸入到一個Fully connected network。這考慮的範圍就是一個`Window`。但如果存在需要考慮整個Sequence才能解決的問題呢?如果將window條大到整個sequence呢?但輸入sequence有長有短，需要統計輸入資料的長度。且sequence太長的話就需要非常多的Fully connected network，那意味著需要非常多的參數，就有可能會有overfitting的問題

## Self-attention
![Self-attention](/2021NTUML/Week5/selfattention.JPG)
self-attention會輸入一整個sequence的向量，輸入多少就輸出個向量，輸出的向量是考慮一整個sequence後得到的

![Self-attention 1](/2021NTUML/Week5/selfattention-1.JPG)
self-attention也能疊加多次

![Self-attention 2](/2021NTUML/Week5/selfattention-2.JPG)
self-attention的輸入可以是輸入的資料或者是hidden layer的輸出

![Self-attention 3](/2021NTUML/Week5/selfattention-3.JPG)
那self-attention要怎麼產生輸出呢?首先會考慮一個輸入向量與其他輸入向量的關聯性

![Self-attention 4](/2021NTUML/Week5/selfattention-4.JPG)
要判斷兩項量的關聯性有多種作法，常見是使用`Dot product`，將兩項量分別乘上不同的矩陣再做Dot product得到關聯性(左)

還有另一種是`Additive`，將兩項量分別乘上不同的矩陣再串接起來，接著計算tanh再乘上一個矩陣得到關聯性(右)

![Self-attention 5](/2021NTUML/Week5/selfattention-5.JPG)
將主向量乘上一個矩陣得到`Query`，其他向量乘上不同的矩陣得到`Key`，將query與不同的key做dot product得到關聯性，也叫做`Attention Score`，一般在實作時也會計算與自己的關聯性。

![Self-attention 6](/2021NTUML/Week5/selfattention-6.JPG)
接著將這些關聯性使用softmax來計算，不一定使用softmax，也可以使用ReLU，常見是使用softmax

![Self-attention 7](/2021NTUML/Week5/selfattention-7.JPG)
有了關聯性的資訊後，就要來抽取重要資訊。將輸入的向量乘上另一個矩陣得到新的v向量，將v向量乘上關聯性資訊後相加起來就是Self-attention輸出的第一個向量。

![Self-attention 8](/2021NTUML/Week5/selfattention-8.JPG)
其他輸出向量也是相同的做法

![Self-attention 9](/2021NTUML/Week5/selfattention-9.JPG)
Self-attention輸出的向量不需要依序產生，他們是同時產生的

![Self-attention 10](/2021NTUML/Week5/selfattention-10.JPG)
![Self-attention 11](/2021NTUML/Week5/selfattention-11.JPG)
![Self-attention 12](/2021NTUML/Week5/selfattention-12.JPG)
![Self-attention 13](/2021NTUML/Week5/selfattention-13.JPG)
將剛剛的流程透過矩陣運算的角度來計算如上四張圖所示

![Self-attention 14](/2021NTUML/Week5/selfattention-14.JPG)
輸入I，經過一連串矩陣運算得到輸出O。整個計算需要機器學習的參數只有Wq、Wk與Wv，這三個參數是未知的

## Multi-head Self-attention
![Multi-head Self-attention 1](/2021NTUML/Week5/multihead-selfattention-1.JPG)
Multi-head Self-attention也就是有多個q、k與v，來計算不同種類的相關性。上圖舉計算2 heads的第一個head為例

![Multi-head Self-attention 2](/2021NTUML/Week5/multihead-selfattention-2.JPG)
第二個head也是相同的做法

![Multi-head Self-attention 3](/2021NTUML/Week5/multihead-selfattention-3.JPG)
計算完兩個head後，再接起來乘上W得到考慮兩個種類的輸出

## Positional Encoding
![Positional Encoding](/2021NTUML/Week5/posencoding.JPG)
Self-attention layer少了位置的資訊，輸入向量的Sequence位置沒有差別。那會有什麼問題呢?比如說做詞性判別，動詞出現再句首的機率低，那位置資訊就顯得重要。因此也可以將位置資訊丟入Self-attention，要使用`Positional Encoding`的技術，將每個位置設計一個專屬的向量，將他加入輸入資料即可。這些位置向量是自行設定，當然也可以透過資料學習得到

## Self-attention for Speech
![Self-attention for Speech](/2021NTUML/Week5/speech.JPG)
把Self-attention應用到語音上，但需要做一些改動，因為一段語音資料往往很長，那會造成什麼問題?
計算Attention matrix的複雜度是輸入向量長度L的平方，需要做L * L次的dot prodocut，所以需要夠大的記憶體才能存下這個矩陣，因此可以使用`Truncated Self-attention`。Truncated Self-attention做的是不看一整句話，只要一小段落，範圍是自行設定。那為何可以這樣做?這就是取決於對問題的理解，要理解一段話的內容其實只需要部分資訊就可以判斷

## Self-attention for Image
![Self-attention for Image](/2021NTUML/Week5/image.JPG)
Self-attention也可以應用在影像上，假設有張圖片是5 * 10 * 3的圖片(3代表RGB)，可以把每個pixel看成一個3維向量，那整張圖片其實是由5 * 10個3維向量組成的

![Self-attention for Image 1](/2021NTUML/Week5/image-1.JPG)
以上是Self-attention應用在影像上的實例

## Self-attention v.s. CNN
![Self-attention v.s. CNN](/2021NTUML/Week5/vs.JPG)
使用Self-attention的話，是考慮一個pixel與其他pixel的關聯性，也就表示考慮整張圖片的資訊。而CNN只考慮Receptive field中的資訊而已。因此CNN是簡化版的Self-attention。CNN的Receptive field是人自行設定的。對Self-attention而言，使用Self-attention來找出相關的pixel，就好像是Receptive field是自動被學出來的，模型決定Receptive field該長什麼樣子

![Self-attention v.s. CNN 1](/2021NTUML/Week5/vs-1.JPG)
這個論文會透過數學的形式來講解CNN是Self-attention的特例，Self-attention只要設定適合的參數也可以做到跟CNN相同的事

![Self-attention v.s. CNN 2](/2021NTUML/Week5/vs-2.JPG)
Self-attention比CNN彈性許多，比較有彈性的模型需要有更多的資料來避免overfitting。透過上圖可以觀察到此現象，因為CNN限制較多，因此資料少時CNN表現比Self-attention好，但當資料量增加時，Self-attention可以得到CNN更好的結果

## Self-attention v.s. RNN
![Self-attention v.s. RNN](/2021NTUML/Week5/vs-3.JPG)
比較兩架構可以得知，Self-attention可以把頭向量考慮到尾向量，如果RNN要做到一樣的事需要頭向量記憶到最後才行。且RNN需要計算完前一個向量才能計算接下來的，但Self-attention可以平行產生所有輸出。Self-attention比RNN有效率，因此現在RNN的架構多數都被Self-attention取代

想了解RNN可以觀看以下連結:
* Recurrent Neural Network : [Video](https://youtu.be/xCGidAeyS4M)

## Self-attention for Graph
![Self-attention for Graph](/2021NTUML/Week5/graph.JPG)
Self-attention也可以應用在graph，但有些特別之處，將node的資訊向量化後還有edge的資訊，可以透過edge的資訊可以知道向量之間的關聯性，因此不需要機器自動找出來，Self-attention只需要計算有相連向量的分數就好，沒有相連的向量分數即為0。將Self-attention結合graph的資訊即為GNN的技術

想了解GNN可以觀看以下連結:
* Graph Neural Network : [Video 1](https://youtu.be/eybCCtNKwzA) [Video 2](https://youtu.be/M9ht8vsVEw8)

## To Learn More
![To Learn More](/2021NTUML/Week5/more.JPG)
由於Self-attention需要非常大的計算量，因此有許多變形的Self-attention來減少運算量，但犧牲的就是模型的表現，雖然表現差一些但運算速度提升許多
* Long Range Arena: A Benchmark for Efficient Transformers [Link](https://arxiv.org/abs/2011.04006)
* Efficient Transformers: A Survey [Link](https://arxiv.org/abs/2009.06732)


