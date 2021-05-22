---
title: "NTU - 機器學習 Week 11 - Adversarial Attack"
date: 2021-05-15T20:36:57+08:00
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

![cover](/2021NTUML/Week11/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/xGQKhbjrFRk) [`Video 2`](https://youtu.be/z-Q9ia5H2Ig)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=xw6K4naFWFg) [`Video 2`](https://www.youtube.com/watch?v=kRmBiV2X810)

## Motivation 
![Motivation](/2021NTUML/Week11/motivation.JPG)
要把訓練的模型應用在現實中，光是正確率高是沒用的，還需要應付人類惡意的攻擊，對於這些惡意行為要能偵測出來

## How to Attack
![How to Attack](/2021NTUML/Week11/attack.JPG)
首先講如何攻擊，假設在做影像辨識，將未受到攻擊的圖片(Benign Image)輸入製模型中，模型能判別說這是貓，那將未受攻擊的圖片加入一些雜訊(通常很小)變成受攻擊的圖片(Attacked Image)，模型則會判別成其他東西，若只是使模型判別變成其他類別是Non-targeted attack，若可以使模型誤判成特並的類別則是Targeted attack

![How to Attack 1](/2021NTUML/Week11/attack-1.JPG)
用ResNet50這個有名的模型來做示範，Benign Image輸入時能辨識成貓咪並有0.64的信心分數，但加入一個人們無法辨視的雜訊後再輸入製模型，發現被辨識成海星，且信心分數達100%

![How to Attack 2](/2021NTUML/Week11/attack-2.JPG)
為了證明兩圖片是不同的，將兩張圖片做相減並放大50倍，可以發現雜訊

![How to Attack 3](/2021NTUML/Week11/attack-3.JPG)
甚至也能加入其他雜訊使貓咪被辨識成鍵盤

![How to Attack 3](/2021NTUML/Week11/non-target.JPG)
至於實際上是怎麼加入雜訊的?首先舉Non-targeted為例，將隨機雜訊加入圖片丟入模型產生的輸出要與預期輸出的差值越大越好，影像辨識的Loss function常使用Cross-entropy，所以在前面加上負號，當這個值越小表示，預測值與實際值差越大

![How to Attack 3](/2021NTUML/Week11/target.JPG)
對於Targeted，加入雜訊的圖片輸入模型得到的輸出值，除了要與預期輸出的差值越大越好，還有與目標值越小越好，因此Loss function會再加上輸出值與目標的Cross-entropy。不管對Non-targeted還是Targeted，接下來都是解這個Optimization的問題，但除了解這問題之外還會加一個限制，為了使加入雜訊的圖片與原圖片看起來非常相似，所以加入的限制就是原圖片與加入雜訊的圖片的差值需要小於某個值，這個值是根據人類的感知來決定

![Non-perceivable](/2021NTUML/Week11/non-perceivable.JPG)
常見有兩個差值的計算，第一種是`L2-norm`，就是將圖片每一維的差值取平方相加，另一種是`L-infinity`，將每一維差值取絕對值，並將最大的當輸出值。至於哪種在攻擊中比較好，假設有張只有4個pixel的圖片，做兩個處理，一個是每個pixel都改一點點，另一個是在右下角的pixel改非常多，這兩種都擁有相同的L2-norm，但在L-infinity卻不同，因此L-infinity比較符合人類的感知能力，但目前是舉影像辨識，若是做聲音訊號的攻擊，則需要研究哪種比較符合人類的聽覺系統

![Approach](/2021NTUML/Week11/approach.JPG)
至於要怎麼解Optimization問題?與先前相同，只是改變的不是模型內參數而是輸入的x，先不考慮限制，初始值也不是隨機給而是原始圖片，接著開始計算Gradient到結束。那考慮限制呢?

![Approach 1](/2021NTUML/Week11/approach-1.JPG)
考慮限制的話，在更新參數後發現不符合限制，則修改更新後的參數，假設使用L-infinity，則x只能在藍色方形中，若在範圍外則在藍色框框上找距離範圍外的點最近的點就好

## FGSM
![FGSM](/2021NTUML/Week11/fgsm.JPG)
接下來介紹一個簡單的攻擊方法Fast Gradient Sign Method (FGSM)，它只需更新一次參數，但在梯度上有個特殊的設計，對梯度每個維度皆取Sign，若大於0則輸出1，小於0輸出-1，接著再乘上現制的值就是更新後的參數，這些參數會落在藍色框框的四個角上

想了解更多FGSM可參考以下連結 : 
* Explaining and Harnessing Adversarial Examples : [Link](https://arxiv.org/abs/1412.6572)

## IFGSM
![IFGSM](/2021NTUML/Week11/ifgsm.JPG)
若FGSM多做幾次就是Iterative FGSM(IFGSM)，但有可能會跑出限制外，所以需將跑出的參數修正回來

## White Box v.s. Black Box
![White Box v.s. Black Box](/2021NTUML/Week11/box.JPG)
先前介紹的都是`White Box Attack`，因為需要知道模型的參數才能計算Gradient，所以對於未知模型參數(如線上API服務)的攻擊就是`Black Box Attack`

## Black Box Attack
![Black Box Attack](/2021NTUML/Week11/blackbox.JPG)
對於未知的模型，假設知道未知模型是用什麼資料訓練出來的，那只要使用相同的資料訓練一個Proxy network，就會有一定的相似程度，所以攻擊Proxy network成功後的圖片丟入未知的模型就會有一定的成功率。那不知道訓練資料呢? 只需要準備一大堆的資料，丟入未知模型看會輸出什麼，再把這些圖片與模型輸出的資訊來訓練一個Proxy network即可

![Black Box Attack 1](/2021NTUML/Week11/blackbox-1.JPG)
上表示單一模型對單一模型的攻擊，對角線因為是相同模型的攻擊，因此是White Box Attack，攻擊成功率100%，對於非對角線的部分則是Black Box Attack。如果像下表使用Ensemble Attack的話，
如-ResNet-152表示使用除了ResNet-152以外的四個模型來攻擊，因此對於下表反而非對角線是White Box Attack，對角線是Black Box Attack，使用Ensemble Attack使得攻擊成功率大幅提升。透過剛剛的表格可以發現Black Box Attack成功非常容易，那為什麼攻擊A模型對於B模型也能成功?這仍然是個待研究的議題，接下來講解普遍相信的結論

有興趣可以參考這篇論文 :
* DELVING INTO TRANSFERABLE ADVERSARIAL EXAMPLES AND BLACK-BOX ATTACKS : [Link](https://arxiv.org/pdf/1611.02770.pdf)

![Black Box Attack 2](/2021NTUML/Week11/blackbox-2.JPG)
上圖每個模型的圖表，橫軸為攻擊該模型可以成功的方向，縱軸是任意方向，而圖表上深藍色的區域為模型會辨識出小丑魚的區域。可以發現每個模型的深藍色區域都蠻類似的，因此攻擊A模型對於B模型也能成功。而這篇論文的結論是攻擊會成功，不是模型的問題而是資料，因為資料特徵的分布使得攻擊不同模型，模型學到差不多的特徵因此才容易成功

有興趣可以參考這篇論文 :
* Adversarial Examples Are Not Bugs, They Are Features : [Link](https://arxiv.org/abs/1905.02175)

## One Pixel Attack
![One Pixel Attack](/2021NTUML/Week11/pixel.JPG)
那攻擊能到多麼細微呢? 有人可以做到`One Pixel Attack`，只更改圖片的一個pixel

有興趣可以參考這篇論文 :
* One pixel attack for fooling deep neural networks : [https://arxiv.org/abs/1710.08864]

## Universal Adversarial Attack
![Universal Adversarial Attack](/2021NTUML/Week11/universal.JPG)
甚至有人可以做到用一個特定的雜訊來攻擊所有類別，使影像辨識系統所有類別都辨識錯誤

有興趣可以參考這篇論文 :
* Universal adversarial perturbations : [Link](https://arxiv.org/abs/1610.08401)

## Attack in the Physical World
![Attack in the Physical World](/2021NTUML/Week11/real.JPG)
剛剛介紹的方法都是在虛擬世界進行，要在現實中進行Attack的話，需要駭進系統為每張圖片加入雜訊嗎?實際上可以不用，有人發明一個特殊的眼鏡，帶上它的人會被人臉辨識系統辨識成知名的藝人，但有些人可能會有些疑慮，如現實是3D世界會有角度問題，但剛剛也提到是可以做到Universal Adversarial Attack，且經過實測從各個角度看這個人都會被辨識錯誤。且論文中也探討許多應用在現實中的各種問題，如攝像頭解析度，會不會雜訊太小使攝像頭偵測不到，或者是電腦與實際做出的成品的色差不同等等

有興趣可以參考這篇論文 : 
* Accessorize to a Crime: Real and Stealthy Attacks on State-of-the-Art Face Recognition : [Link](https://www.cs.cmu.edu/~sbhagava/papers/face-rec-ccs16.pdf)

![Attack in the Physical World 1](/2021NTUML/Week11/real-1.JPG)
除了人臉辨識，由於自駕車發展快速，有人也對車牌辨識等進行攻擊，將一些號誌貼上貼紙使系統辨識錯誤

![Attack in the Physical World 2](/2021NTUML/Week11/real-2.JPG)

但這些貼紙太招搖，容易被人發覺，因此有人去改變號誌字體的特徵，使其比較不招搖一些，向上圖限速35，把3拉長一點系統會辨識成限速85，下面有DEMO影片

{{< youtube 4uGV_fRj0UA>}}

## Adversarial Reprogramming
![Adversarial Reprogramming](/2021NTUML/Week11/repro.JPG)
攻擊甚至能改變模型的行為，如ImageNet原本是辨識物體的模型，而有人想做一個數圖片中的方塊，但不想自己訓練而是選擇寄生在已有的模型上，當輸入圖片有4個方塊，ImageNet就會輸出tiger shark，而使用者可以知道這張圖片有4個方塊，這樣就可以操控ImageNet來做原本不是它該做的事

有興趣可以參考這篇論文 : 
* Adversarial Reprogramming of Neural Networks : [Link](https://arxiv.org/abs/1806.11146)

## "Backdoor" in Model
!["Backdoor" in Model](/2021NTUML/Week11/backdoor.JPG)
之前的攻擊都是在輸入時加入雜訊，但有些攻擊可能從訓練就開始，在訓練資料中加入被攻擊的圖片，不是正確的圖片標記錯誤這種，而是正確的圖片加入看不見的雜訊。當訓練完成後，輸入其他圖片辨識都正確，但在特定的圖片會發生錯誤。因此使用未知的公開資料集時要小心，雖然目前還有很多限制，但這樣的可能性仍然存在

有興趣可以參考這篇論文 :
* Poison Frogs! Targeted Clean-Label Poisoning Attacks on Neural Networks : [Link](https://arxiv.org/abs/1804.00792)

## Defense
![Defense](/2021NTUML/Week11/defense.JPG)
提了這麼多種的攻擊，接下來介紹如何防禦，防禦有分兩種，一種是`Passive Defense`，另一種是`Proactive Defense`

## Passive Defense
![Passive Defense](/2021NTUML/Week11/passive.JPG)
Passive Defense是不動訓練好的模型，而是將輸入圖片經過濾波，那要做怎樣的濾波呢?其實使用模糊化的方式就可以達到不錯的防禦

![Passive Defense 1](/2021NTUML/Week11/passive-1.JPG)
先前提到加入雜訊的貓咪被辨識成鍵盤，若將這張圖片經過模糊化再去辨識結果就變正確了，理由是攻擊只會在特定的方向上，不是任意一個雜訊都可以攻擊成功，而經過模糊化後能改變攻擊方向，使其失敗。雖然防禦成功，但有副作用，如原本辨識成功的圖片信心分數會下降

![Passive Defense 2](/2021NTUML/Week11/passive-2.JPG)
除了模糊化以外，還有使用會失真的壓縮或者使用Generator來重新繪製圖片，由於微小的雜訊是Generator沒有看過的，因此重新繪製時可能無法復原，因此能防禦

有興趣可以參考以下論文 :
* Feature Squeezing: Detecting Adversarial Examples in Deep Neural Networks : [Link](https://arxiv.org/abs/1704.01155)
* Shield: Fast, Practical Defense and Vaccination for Deep Learning using JPEG Compression : [Link](https://arxiv.org/abs/1802.06816)
* Defense-GAN: Protecting Classifiers Against Adversarial Attacks Using Generative Models : [Link](https://arxiv.org/abs/1805.06605)

![Passive Defense 3](/2021NTUML/Week11/passive-3.JPG)
但Passive Defense有個弱點，就是當攻擊者知道這模型會防禦時，就把攻擊整個加入模糊化的模型，因為模糊化就像是在模型前再加入一層，所以攻擊加入模糊化後的模型就可以躲過這種防禦方式。因此使用Passive Defense時，當攻擊者不知道時防禦效果很強，但知道後就會失去效用。要強化防禦的話就是加入隨機性，比如這篇論文對輸入圖片隨機加入各種圖片處理的方式，如更改大小、隨機填充等，但當攻擊者知道隨機的分布時，就有可能被攻破

有興趣可以參考這篇論文 :
* Mitigating Adversarial Effects Through Randomization : [Link](https://arxiv.org/abs/1711.01991)

## Proactive Defense
![Proactive Defense](/2021NTUML/Week11/proactive.JPG)
另一種防禦是Proactive Defense，會使用`Adversarial Training`的方式，先訓練好一個模型，再去將輸入圖片加入攻擊的雜訊但標記成正確產生一個被攻擊的資料集，將這個資料集拿去重新訓練。簡單來說就是訓練好模型，然後找漏洞，再去補漏洞，是一種Data Augmentation的方法。這樣的防禦缺點是不太能抵擋新的攻擊，因為新的攻擊造成的漏洞是訓練時沒看過的。另一個缺點是需要較大的運算資源，因為一直擴增訓練資料來抵擋攻擊

但有人使用特殊方法來做到Adversarial Training的效果且不需要較大的運算，有興趣可以參考這篇論文 : 
* Adversarial Training for Free! : [Link](https://arxiv.org/abs/1904.12843)

