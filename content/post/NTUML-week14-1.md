---
title: "NTU - 機器學習 Week 14 - Domain Adaptation"
date: 2021-05-24T15:33:26+08:00
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

![cover](/2021NTUML/Week14/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video`](https://youtu.be/Mnk_oUrgppM)

Youtube Video Link (English): [`Video`]

## Domain Shift
![Domain Shift](/2021NTUML/Week14/domain.JPG)
到目前為止已經知道如何訓練許多模型，且大多能獲得極高的正確率，但當測試資料的分布與訓練資料不同那該怎麼辦? 舉訓練手寫數字為例，如果測試資料圖片的背景不再是黑白而是彩色的，可以發現準確率會低很多。這種測試與訓練資料分布不同的叫做`Domain Shift`，要解決這種問題就需要`Domain adaptation`的技術，這種技術可以看成是一種Transfer learning

若對Transfer learning感興趣的可以看此連結影片 : 
* ML Lecture 19: Transfer Learning : [Video](https://youtu.be/qD6iD4TFsdQ)

![Domain Shift 1](/2021NTUML/Week14/domain-1.JPG)
Domain Shift除了輸入分布有變化以外，輸出的分布也可能有變化，可能特定類別輸出的機率特別大，甚至還有一種罕見的可能是輸入輸出分布相同，但關係產生了變化，可能訓練資料的標籤是0，但測試資料標籤變成1。接下來介紹會著重在輸入資料不同的方面，來自訓練資料的會稱為`Source domain`，來自測試資料的會稱為`Target domain`

## Domain Adaptation
![Domain Adaptation](/2021NTUML/Week14/adaptation.JPG)
接下來舉例會需要`Domain Adaptation`的情境，首先會有一堆訓練資料訓練出來的模型，但我們會希望這模型能應用在不同的domain上，因此訓練時會需要這模型對不同的domain有一些的了解，通常會需要一點另一個domain上些許有標記的資料，但通常Target domain的資料量非常少，因此要小心不要overfit

![Domain Adaptation 1](/2021NTUML/Week14/adaptation-1.JPG)
而更常見的情境是擁有大量Target domain的資料，但都沒有標註，那要怎麼用這些沒有標註的資料在Source domain訓練出一個模型可以用在Target domain上呢?

![Domain Adaptation 2](/2021NTUML/Week14/adaptation-2.JPG)
基本的想法是訓練一個`Feature Extractor`來把不同的特徵濾掉，留下Source domain與Target domain共同的特徵，這樣就會有相同的分布，這些特徵可以幫助我們在Source domain訓練一個模型並可以套用在Target domain，那要怎麼得到Feature Extractor呢?

![Domain Adaptation 3](/2021NTUML/Week14/adaptation-3.JPG)
一個Classifier可以看成兩個部分，一部分是Feature Extractor，另一部分是Label Predictor，對於Source domain的資料，訓練輸入的資料與得到的輸出越接近越好，但問題是Target domain沒有標註，所以要比較兩個domain在Feature Extractor後輸出的分布是否類似，為了能讓兩分布越接近，因此要使用`Domain Adversarial Training`的技術

![Domain Adaptation 4](/2021NTUML/Week14/adaptation-4.JPG)
Domain Adversarial Training就是訓練一個二元分類器，輸入這些分布後判斷來自Source domain還是Target domain，所以Feature Extractor有額外的任務要Source domain的label能騙過Domain Classifier，這樣的步驟就像是GAN。Feature Extractor要騙過Domain Classifier只要輸出0就好，但因為Label Predictor需要Feature Extractor來輸出預測的標註，所以Feature Extractor就不會輸出0

![Domain Adaptation 5](/2021NTUML/Week14/adaptation-5.JPG)
在論文中可以發現，使用Target domain訓練的模型直接用在Source domain上後準確率降低很多，加入Domain Adaptation後結果有明顯提升

![Limitation](/2021NTUML/Week14/limit.JPG)
但這樣的方法會有個限制，假設藍色原點與藍色三角形是Source domain的標註，那我們可以找的到一條界線來分開它們，而橘色正方形是Target domain的資料，它們並沒有標註，那上圖哪個的標註才算使Target domain越接近Source domain，顯然是右邊，所以會希望橘色正方形要遠離分界線

![Limitation 1](/2021NTUML/Week14/limit-1.JPG)
那要怎麼使橘色正方形要遠離分界線?輸入這些沒有標記的資料，輸出的類別越集中表示界線越遠，反之輸出的類別越分散表示界線越近。當然還有其他方法，有興趣的可以參考以下論文 : 
* A DIRT-T Approach to Unsupervised Domain Adaptation : [Link](https://arxiv.org/abs/1802.08735)
* Maximum Classifier Discrepancy for Unsupervised Domain Adaptation : [Link](https://arxiv.org/abs/1712.02560)

![Limitation 2](/2021NTUML/Week14/limit-2.JPG)
還有一個問題，目前都是假設Source domain與Target domain的類別是相同的，但Target domain的資料沒有標記，所以不知道它有什麼類別，兩個Domain的交集情況是不知道的，硬做可能使不同的類別分在一塊，要解決這問題可以使用`Universal domain adaptation`，有興趣的可以參考這篇論文 : 
* Universal Domain Adaptation : [Link](https://openaccess.thecvf.com/content_CVPR_2019/html/You_Universal_Domain_Adaptation_CVPR_2019_paper.html)

![Domain Adaptation 6](/2021NTUML/Week14/adaptation-6.JPG)
但如果遇到Target domain的資料沒有標記且很少，那可以使用`Testing Time Training(TTT)`，有興趣的可以參考這篇論文 : 
* Test-Time Training with Self-Supervision for Generalization under Distribution Shifts : [Link](https://arxiv.org/abs/1909.13231)

![Domain Adaptation 7](/2021NTUML/Week14/adaptation-7.JPG)
更嚴峻的情況是對Target domain的資料一無所知，這時就沒辦法做Domain Adaptation，那要怎麼辦?

## Domain Generalization
![Domain Generalization](/2021NTUML/Week14/general.JPG)
對Target domain的資料一無所知的話，可以透過`Domain Generalization`，有分兩種情形，一種是訓練資料非常豐富含有多個Domain，模型可以彌補之間的差異，這情況可以透過下面這篇論文來解決 : 
* Domain Generalization with Adversarial Feature Learning : [Link](https://ieeexplore.ieee.org/document/8578664)

另一種是訓練資料可能就一個Domain，但測試資料有多個Domain的話，簡單概述解法是透過Domain Adaptation產生多個Domain的資料再透過上面的方法來解決，有興趣的可以參考下面這篇論文 : 
* Learning to Learn Single Domain Generalization : [Link](https://arxiv.org/abs/2003.13216)
