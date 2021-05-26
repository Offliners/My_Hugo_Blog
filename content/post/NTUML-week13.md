---
title: "NTU Machine Learning Week 13 - Explainable AI"
date: 2021-05-16T03:42:10+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 13 - Explainable AI

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week13/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/WQY85vaQfTI) [`Video 2`](https://youtu.be/0ayIPqbdHYQ)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=4rVD1EOaAX4) [`Video 2`]

## Why we need Explainable ML?
![Why we need Explainable ML?](/2021NTUML/Week13/issue.JPG)
模型得到正確答案不代表模型很聰明，因此在將來，模型得到正確答案時需要解釋為什麼得到這個答案。以上為現今機器學習應用的舉例，如銀行借貸款、醫療診斷、法律判決與自駕車等，模型需要為得出的答案給出合理的解釋才能使人信服

## Interpretable v.s. Powerful
![Interpretable v.s. Powerful](/2021NTUML/Week13/vs.JPG)
若採用較好解釋的模型如Linear model，但往往表現較差;若使用Deep model較難解釋，但表現較好。因此有人認為不應該使用Deep model，因為它是個黑盒子，但這有點削足適履，應該想辦法去提升模型的解釋力

![Decision tree](/2021NTUML/Week13/tree.JPG)
可能有人會提那使用`Decision tree`? Decision tree相較Linear model強大，也比Deep model好解釋，透過節點問題分析來選擇最終決斷

![Decision tree 1](/2021NTUML/Week13/tree-1.JPG)
但一棵Decision tree也可以有複雜的結構(左圖)，而且實際上也不會只使用一棵，而是`Random forest`的技術(右圖)，使Decision tree變得複雜難以解釋

## Goal of Explainable ML
![Goal of Explainable ML](/2021NTUML/Week13/goal.JPG)
以往模型訓練都有個明確的目標，如降低loss、提升accuracy，那Explainable ML是什麼呢?其實Explainable ML目標非常不明確，許多人會覺得Explainable ML要能闡述模型的一切，模型是怎麼做的，怎麼得到這樣的結果，來打開Deep network這個黑盒子，因為很多人認為Deep network是個黑盒子不能相信，但其實現實生活中充滿著許多黑盒子，如人腦，人腦也是個黑盒子但我們可以相信另一個人的決策，那為什麼不能相信Deep network?也許就差在Deep network沒給出一個令人信服的理由。這裡舉了一個哈佛大學做的一個實驗，大家在排隊等印表機時，有人插隊的理由是只有5頁要印，有60%的人會同意，如果理由是趕時間時，同意的人提高到94%，但神奇的是如果理由是他必須要先印，同意的人也能高達到93%

![Goal of Explainable ML 1](/2021NTUML/Week13/goal-1.JPG)
因此Explainable ML的目標就是要給出使人信服的理由，對象可能是你的客戶、老闆甚至是自己

## Explainable ML
![Explainable ML](/2021NTUML/Week13/explainableML.JPG)
Explainable ML分成兩大類，一類是`Local Explanation`，比如影像辨識說這張圖片是一隻貓，那機器要回答說為什麼這張圖片是一隻貓，根據這張圖片來回答。另一個是`Global Explanation`，要問機器說什麼樣的圖片他會覺得是一隻貓，並不是針對任一張圖片

## Local Explanation : Explain the Decision
![Local Explanation](/2021NTUML/Week13/local.JPG)
首先講述Local Explanation，一張圖片機器覺得是貓是根據什麼?可能根據圖片的組成元素，看哪些元素對機器的決策是較重要的。那要怎麼判斷呢?基本原則是將每個元素移除或修改，看機器輸出的變化，如果有個元素移除後輸出變化較大，表示這個元素是較重要的

![Local Explanation 1](/2021NTUML/Week13/local-1.JPG)
簡單的實作方法如上圖，將圖片中置入一個灰色方塊並移動到圖片的任意位置來看機器的輸出，如果是藍色表示輸出正確的機率是低的，紅色是高的，以狗為例，可以發現當灰色遮擋到狗狗的臉的話，機器判斷正確的機率會降低，表示這部分是較重要的元素

![Local Explanation 2](/2021NTUML/Week13/local-2.JPG)
進階的作法是，將圖片丟入模型看輸出的loss，接著將圖片中的每個pixel加上一些小變化觀察輸出的loss，若loss變化很大表示這個pixel是較重要的。將每個loss變化除以pixel變化取絕對值得到比值，也就是將loss對每個pixel取偏微分，就可以繪製出`Saliency Map`，在Saliency Map中較白色的部分表示比值越大，代表這個位置的pixel較重要

![Case Study: Pokémon v.s. Digimon](/2021NTUML/Week13/case.JPG)
接下來舉分類神奇寶貝與數碼寶貝為例

![Case Study: Pokémon v.s. Digimon 1](/2021NTUML/Week13/case-1.JPG)
任務是訓練一堆神奇寶貝與數碼寶貝的圖片並進行分類，再將模型沒看過的圖片輸入後來判斷這圖片是神奇寶貝還是數碼寶貝

![Case Study: Pokémon v.s. Digimon 2](/2021NTUML/Week13/case-2.JPG)
模型使用簡單的3層CNN，實驗結果發現訓練的accuracy高達98.9%，測試的accuracy居然也有98.4%

![Case Study: Pokémon v.s. Digimon 3](/2021NTUML/Week13/case-3.JPG)
上圖是數碼寶貝的Saliency Map

![Case Study: Pokémon v.s. Digimon 4](/2021NTUML/Week13/case-4.JPG)
上圖是神奇寶貝的Saliency Map

![Case Study: Pokémon v.s. Digimon 5](/2021NTUML/Week13/case-5.JPG)
比較後發現亮點都在背景上，而不是寶貝的特徵上，為什麼呢?原來是因為神奇寶貝的圖片都是PNG檔，輸入後背景都是黑色的，因此機器只要判斷背景就知道是神奇寶貝還是數碼寶貝了

![Case Study: Pokémon v.s. Digimon 6](/2021NTUML/Week13/case-6.JPG)
可能有人覺得判斷神奇寶貝與數碼寶貝的例子很荒謬，但現實中也有可能發生，在某個公開數據集中使用Saliency Map來看機器是怎麼判斷馬，發現都在左下角的文字，原來這個數據集中馬的圖片都來自某個網站，都會有浮水印，因此機器只要看到這串文字就知道是馬。以上例子可以凸顯Explainable ML的重要性

## Limitation: Noisy Gradient
![Limitation: Noisy Gradient](/2021NTUML/Week13/smoothgrad.JPG)
那樣怎麼把Saliency Map畫得更好，可以透過`SmoothGrad`，由於一般的Saliency Map雖然可以標出重要特徵，但周圍仍會有一些雜訊，使用SmoothGrad會加入各種不同雜訊後分別計算Saliency Map再平均起來，就會凸顯出重要特徵。有人會問說一般的Saliency Map可能周圍也是重要特徵等，這就要看怎樣的結果能使人信服，顯然SmoothGrad後的Saliency Map凸顯得特徵較能說服人說這是一隻蹬羚

## Limitation: Gradient Saturation
![Limitation: Gradient Saturation](/2021NTUML/Week13/limit.JPG)
但光比較Gradient沒辦法完全反映一個元素的重要性，假設要判斷圖片是否是大象，可以從鼻子的長度下手，但鼻子當長到一個程度後就不會覺得說圖片更像是一隻大象，這時候的偏微分會趨近於0，因此在Gradient或者Saliency Map會發現鼻子得長度對判斷大象是不重要的，但事實不是如此，要解決這問題可以透過`Integrated gradient`，想了解更多可以參考這篇論文 : 
* Gradients of Counterfactuals : [https://arxiv.org/abs/1611.02639]

## How a network processes the input data?
![How a network processes the input data?](/2021NTUML/Week13/input.JPG)
以上都是看輸入的哪些部份比較重要，接下來要看Network怎麼處理這些輸入，最直覺的方法是直接人眼去看，假設每個Layer都有100個neuron，可以看成100維的向量，可以透過分析這些向量來了解Network中發生什麼事，但高維向量不好觀察與分析，所以可以透過PCA或者t-SNE來降維

![How a network processes the input data? 1](/2021NTUML/Week13/input-1.JPG)
舉語音辨識為例，左圖為資料集中的特徵，不同的人說同樣的話看起來特徵十分凌亂，但經過8層Layer後發現特徵會聚集成一條線，每條線代表同內容的話

![How a network processes the input data? 2](/2021NTUML/Week13/input-2.JPG)
除了使用Lyaer的輸出向量來做Explanation，也可以透過Attention來做，但會有些論文覺得Attention無法做Explanation，有些覺得可以，像以下兩篇 :  
* Attention is not Explanation : [Link](https://arxiv.org/abs/1902.10186)
* Attention is not not Explanation : [Link](https://arxiv.org/abs/1908.04626)

Attention在什麼情況中可以什麼情況中不行，仍是尚待研究的議題

![How a network processes the input data? 3](/2021NTUML/Week13/probing.JPG)
因為肉眼判斷仍有極限，可能有些現象沒觀察到，所以可以訓練`Probing`，也就是個分類器，例如訓練 個POS的分類器，將BERT中的向量輸入來判斷是什麼詞性的字彙，或者訓練一個NER分類器來判斷這個詞彙是地名還是人名等等。但要注意分類器的強度，有時候正確率低不代表這些向量沒有好的資訊，有可能是分類器本身就不強

## Global Explanation : EXPLAIN THE WHOLE MODEL
![Global Explanation](/2021NTUML/Week13/global.JPG)
Global Explanation不會針對特定的輸入做Explanation。假設有一張圖片輸入至CNN模型，如果發現Filter 1有很多neuron有較大的值，表示Filter 1看到許多重要的特徵。但要使用Global Explanation的話，就不能針對這張圖片，而是要探討Filter 1想要在圖片上看到什麼樣的特徵，那要怎麼做?就產生一張圖片包含所有Filter 1想看到的特徵，這樣就可以看這張圖片來了解Filter 1想要偵測到什麼特徵

![Global Explanation 1](/2021NTUML/Week13/global-1.JPG)
那要怎麼找這張圖片呢?輸入未知圖片X經過Filter 1後所得到的值要越大越好，這張圖片就是我們要的，解這樣的問題就會變成Gradient Ascent。觀察這張圖片就可以了解Filter 1想偵測的特徵

![MNIST](/2021NTUML/Week13/case-7.JPG)
上圖為實際CNN模型做MNIST訓練後，把第二層卷積層的Filter做Global Explanation後的結果，可以發現每個Filter都有它想偵測的筆畫方向

![MNIST 1](/2021NTUML/Week13/case-8.JPG)
那如果看最後一層做Global Explanation的結果呢?會是0~9的圖形嗎?最後一層的輸出會是0~9的機率分布，假設想看輸出1的話，則將未知圖片X輸入模型，看怎樣的X能使輸出1的機率最高。依此類推做0~9後發現每個X都不是0~9的形狀，而是一堆雜訊，為什麼看到這些雜訊後CNN會覺得是數字?因為知道adversarial attack，我們知道即使輸入雜訊，機器也能辨識出一些東西，但這沒辦法給出令人信服的答案，要怎麼辦呢?

![MNIST 2](/2021NTUML/Week13/case-9.JPG)
答案是加入一些限制，輸入的未知圖片X除了使模型輸出特定數字的機率最大以外還要有一個限制，就是這張圖片有多像是該數字，像上右圖加入的限制是圖片X的白點越少越好，但仍不像數字，因為加入限制與一些超參數調整不是那麼容易

![Case](/2021NTUML/Week13/case-10.JPG)
像上圖是對影像辨識模型下了許多功夫、限制與超參數的調整才達到比較好的結果，這需要取決於對影像的了解程度

有興趣可以參考這篇論文 : 
* Understanding Neural Networks Through Deep Visualization : [Link](https://arxiv.org/abs/1506.06579)

## Constraint from Generator
![Constraint from Generator](/2021NTUML/Week13/generator.JPG)
若需要產生較清楚圖片的話，可以使用Generator的技術來找雜訊z，再透過Generator來產生未知圖片X。將Generator與Classifier接在一起，輸入雜訊z後得到圖片，這圖片會使Classifier輸出特定類別的機率越高越好，這樣就可以將z輸入給Generator看圖片X長什麼樣子

![Constraint from Generator 1](/2021NTUML/Week13/generator-1.JPG)
透過這樣的技術，可以找到較符合我們想像的圖片。但有些人可能覺得這有點太強調，因為原本機器看到的與我們想看到的不同，因此硬是用一些方法來改變。因為Explainable ML往往會使用合理的方法來將機器所想的來接近人們想看到的，使人們得到較信服的理由

## Outlook
![Outlook](/2021NTUML/Week13/lime.JPG)
Explainable ML除了Local Explanation與Global Explanation，還有個技術是使用較簡單的模型想辦法來模仿複雜模型的行為，假設能模仿那就分析簡單模型的行為就好，假設使用Linear model來模仿複雜模型成功的話，那可以分析Linear model來了解複雜的黑盒子，但會存在許多問題像是簡單模型真的能模仿複雜的模型嗎?當然是不行，因此有個經典的方法`LIME`，只模仿一小區塊的行為而已

若想更了解Lime可以參考以下影片 : 
* Using a Model to Explain Another : [Video](https://youtu.be/K1mWgthGS-A)
* Lime : [Video](https://youtu.be/OjqIVSwly4k)

