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

Youtube Video Link (Chinese): [Video 1](https://youtu.be/4OWp0wDu6Xw) [Video 2](https://youtu.be/jNY1WBb8l4U) [Video 3](https://www.youtube.com/watch?v=MP0BnVH2yOo) [Video 4](https://youtu.be/wulqhgnDr7E)

Youtube Video Link (English): [Video 1](https://www.youtube.com/watch?v=Mb9kddLfLRI) [Video 2](https://www.youtube.com/watch?v=kFhv1I_fbZI) [Video 3] [Video 4]

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

GAN是以很難訓練聞名，很多人認為是因為沒有Minimize JS Divergence，上述的論文提告有方法可以Minimize JS Divergence，但即使做到這樣表現還是沒有很好

## JS divergence is not suitable
![JS divergence is not suitable](/2021NTUML/Week7/js.JPG)
大多數的情況，隨機的分布與資料實際的分布重疊部分非常少，理由有兩個 : 
1. 由於這兩個分布都是要產生圖片，但圖片其實是高維空間中低維度的manifold，在高維空間中隨便sample一個點通常無法構成一個二次元人物的頭像，因此從資料特性來看，他們都像二維空間的兩條線，除非剛好重合，否則相交的範圍幾乎可以忽略
2. 我們從來不知道這兩個的分布範圍長什麼樣子，可能重合部分很多，但因為實際都是透過sample，因此如果sample不夠多不夠密，那從分布來看還是沒有重疊

![JS divergence is not suitable](/2021NTUML/Week7/js-1.JPG)
如果兩分布幾乎沒有重疊的話，那JS divergence計算出來會是log2，因此不管兩分布距離多遠，只要沒有重合就會是log2。距離遠近都是log2使得訓練更新參數時無法使Generator產生的新分布靠實際分布更近。實際操作來看，使用JS divergence時，訓練Binary Classifier分辨圖片的正確率會是100%，因為sample的圖片太少，Discriminator容易分辨出來

## Wasserstein distance
![Wasserstein distance](/2021NTUML/Week7/wasserstein.JPG)
由於JS divergence有上述問題，因此有人使用`Wasserstein distance`來計算分布的平均距離

![Wasserstein distance 1](/2021NTUML/Week7/wasserstein-1.JPG)
但對於複雜的分布，計算Wasserstein distance會非常困難，假設有P、Q兩種不同的分布，要把P堆成像Q就有很多種方法，因此就有許多不同的平均距離，那就有了一個定義，窮舉所有可能的方法並計算平均距離，最短的就是Wasserstein distance

![Wasserstein distance 2](/2021NTUML/Week7/wasserstein-2.JPG)
先不管計算Wasserstein distance有多複雜，Wasserstein distance能使兩分布距離縮短，就能使Generator進步

## WGAN
![WGAN](/2021NTUML/Week7/wgan.JPG)
當使用Wasserstein distance取代JS divergence時的GAN就是`WGAN`，藉由解上述Optimization問題得到的值就是Wasserstein distance。這裡還有個限制，就是D function必須是`1-Lipschitz function`，也就是要足夠平滑。值觀來想，若沒有平滑的限制，會使兩邊的期望值趨近無限大的正值與無限大的負值，使訓練無法收斂

![WGAN 1](/2021NTUML/Week7/wgan-1.JPG)
那要怎麼做到這個限制? 一開始的WGAN，將訓練的weight限制在一定的範圍內，但這方法不一定能達到這限制。因此之後提了許多方法來更好訓練WGAN，想了解更多可參考以下連結:
* Improved Training of Wasserstein GANs : [Link](https://arxiv.org/abs/1704.00028)
* Spectral Normalization for Generative Adversarial Networks : [Link](https://arxiv.org/abs/1802.05957)

## GAN still challenging
![GAN still challenging](/2021NTUML/Week7/challenge.JPG)
當其中一個模型發生問題而停止訓練，那另外一者也會停止或表現更差。因此有許多訓練GAN的技巧，有興趣的可以參考以下連結 : 
* How to Train a GAN? Tips and tricks to make GANs work : [Link](https://github.com/soumith/ganhacks)
* Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks : [Link](https://arxiv.org/abs/1511.06434)
* Improved Techniques for Training GANs : [Link](https://arxiv.org/abs/1606.03498)
* Large Scale GAN Training for High Fidelity Natural Image Synthesis : [Link](https://arxiv.org/abs/1809.11096)

## GAN for Sequence Generation
![GAN for Sequence Generation](/2021NTUML/Week7/gan-seq2seq.JPG)
訓練GAN最困難的在生成文字，因為會發現無法透過梯度下降法來使得分更高，因為decoder的微小變化無法使輸入Discriminator的token造成改變，因此得分也不變，就沒辦法算微分。那可能有人會問CNN有maxpooling可以算梯度，那為什麼GAN在decoder不能?這問題就留給大家深思

那不能使用梯度下降法怎麼辦?使用`Reinforcement Learning (RL)`硬做

![GAN for Sequence Generation 1](/2021NTUML/Week7/gan-seq2seq-1.JPG)
RL是以難訓練聞名，GAN也是以難訓練聞名，因此兩個結合就更難訓練，因此有很長的一段時間都沒有人可以成功使用GAN來訓練Generator生成文字，通常需要pretrain

![GAN for Sequence Generation 2](/2021NTUML/Week7/gan-seq2seq-2.JPG)
直到有一篇文章使用Scratch的方式，並使用一大堆的超參數與技巧，有興趣的可以參考以下連結 : 
* Training language GANs from Scratch : [Link](https://arxiv.org/abs/1905.09922)

想更了解GAN或其他Generative模型的可以參考以下連結 : 
*  Generative Adversarial Network (GAN) : [Video List](https://www.youtube.com/playlist?list=PLJV_el3uVTsMq6JEFPW35BCiOQTsoqwNw)
*  Unsupervised Learning - Deep Generative Model : [Video](https://youtu.be/8zomhgKrsmQ) 
*  Flow-based Generative Model : [Video](https://youtu.be/uXY18nzdSsM)

## Other Method
![Possible Solution](/2021NTUML/Week7/supervise.JPG)
除了使用GAN來生成，那能不能使用Supervised Learning來做? 可以，將每個圖片配置向量，將這項量輸入到network中，輸出就是這個向量對應的圖片。但難的點在於如果圖片對應的向量是隨機的表現會很差，需要特殊的方法來配置向量，對特殊方法有興趣的可以參考以下連結 : 
* Optimizing the Latent Space of Generative Networks : [Link](https://arxiv.org/abs/1707.05776)
* Gradient Origin Networks : [Link](https://arxiv.org/abs/2007.02798)

## Evaluation of Generation
### Quality of Image
![Quality of Image](/2021NTUML/Week7/quality.JPG)
那要怎麼評估GAN生成的東西是好還是壞?最直覺的作法是找人來評量，但這會有不客觀與不穩定的問題。針對特定的任務可以有特定的方法，如動漫人物生成，可以透過訓練人臉偵測系統來捕捉GAN生成的人臉是否成功。對一般的圖片生成case，可以透過訓練影像分類系統，將生成圖片來做分類，如果越集中在特定的類別，那表示這個GAN表現得越好

### Mode Collapse
![Mode Collapse](/2021NTUML/Week7/mode.JPG)
透過上述方法還是行不通的，因為會有`Mode Collapse`的問題，也就是生成的圖片就只有特定的幾張，如動漫人物生成，可以發現同樣的臉出現機率變高。那為什麼會有這樣的問題? 因為當Generator生成特定圖片能一直騙過Discriminator，因此Generator就提高產生出這圖片的機率。那要怎麼避免呢? 至今還沒有很好的方法來完全避免，最有效的方法就是存取每次訓練的checkpoint，當發現遇到Mode Collapse時，就停止訓練去尋找還沒有Mode Collapse時的checkpoint來用

### Mode Dropping
![Mode Collapse 1](/2021NTUML/Week7/mode-1.JPG)
除了Mode Collapse的問題，還有更難被偵測到的`Mode Dropping`，Mode Dropping是指產生的資料只有真實資料的一部分，單純看產生的資料會覺得不錯且多樣性夠，但其實真實資料的多樣性更大。至今有名的模型可能也都還有Mode Dropping問題，當看很多生成的圖片後，就可以猜出說這是生成的人臉

### Diversity
![Diversity](/2021NTUML/Week7/diversity.JPG)
那要怎麼知道模型多樣性夠不夠? 有作法是將生成的圖片丟入Classifier來做分類，每個圖片可以獲得一個分布，將所有分布平均起來來判斷，如果分布集中表示多樣性不夠

![Diversity 1](/2021NTUML/Week7/diversity-1.JPG)
反之，如果分布平均表示多樣性足夠。有些人可能覺得Quality與Diversity有點互斥，但他們評估範圍不同，Quality是只看一張圖片，Diversity是看一堆圖片。因此有個指標是`Inception Score (IS)`，如果Quality高，Diversity大，IS就會越高分。但對於特定類別的生成，如二次元人臉生成，因為輸出都是人臉，Quality都會集中在人臉的類別因此高，但Diversity就會較集中因此低，所以IS可能不適用

### Fréchet Inception Distance (FID)
![Fréchet Inception Distance](/2021NTUML/Week7/FID.JPG)
FID是將圖片輸入Classifier，拿取經過softmax前產生類別前的高維度向量，因為就算類別都是人臉，但向量的分布還是會不同。將實際資料的分布與生成向量的分布都假設是高斯分布，然後計算`Fréchet Distance`，因為是距離，所以值是越小越好。但會有些問題，如分布不是高斯分布、準確計算向量分布需要大量的樣本資料等

想了解Fréchet Distance可以參考以下連結 : 
* GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium : [Link](https://arxiv.org/pdf/1706.08500.pdf)

![Fréchet Inception Distance 1](/2021NTUML/Week7/FID-1.JPG)
有實驗將四個不同資料集與不同的模型分別去計算FID，因為有不同的Random seed去訓練，因此得到的FID是一個分布，而不是單一數值。其中還有個不是GAN的作法是VAE，明顯可以看出GAN產生的結果比VAE更好，因為FID是越小越好。有些人可能覺得不同的GAN結果差不多，是因為這個實驗的network架構都是同一個，實際上可能某些network架構對某種GAN比較好

對此實驗有興趣可以參考以下連結 : 
* Are GANs Created Equal? A Large-Scale Study : [Link](https://arxiv.org/abs/1711.10337)

### We don't want memory GAN
![We don't want memory GAN](/2021NTUML/Week7/memory.JPG)
除了上述的評估方法，GAN還有可能有個問題沒辦法被評估，假設Generator能產生出跟真實資料相同的圖片，FID會很小，但如果GAN只會產生相同圖片不如直接去資料集中sample圖片就好，何必用GAN? 有人可能會提說比較生成圖片與真實圖片的相似度，那如果GAN只會產生左右翻轉的圖片呢? 相似度會比不出來

若有興趣了解GAN的評估方式可參考以下連結，這篇論文有20種評估GAN Generator的方式 : 
* Pros and Cons of GAN Evaluation Measures : [Link](https://arxiv.org/abs/1802.03446)