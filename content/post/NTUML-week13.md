---
title: "NTU - 機器學習 Week 13 - Explainable AI"
date: 2021-05-16T03:42:10+08:00
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


