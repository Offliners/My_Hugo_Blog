---
title: "NTU Machine Learning Week 17 - Life-long Learning"
date: 2021-06-19T14:44:06+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 17 - Life-long Learning

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week17/cover-1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/rWF9sg5w6Zk) [`Video 2`](https://youtu.be/Y9Jay_vxOsM)

Youtube Video Link (English): [`Video 1`](https://youtu.be/yAX8Ydfek_I) [`Video 2`](https://youtu.be/-2r4cqDP4BY)

## Life Long Learning in real-world applications
![Life Long Learning](/2021NTUML/Week17/lll.JPG)
`Life Long Learning`在現實生活中可以應用在線上系統，當用戶用完後提供回饋，讓模型有新的資料與新任務繼續訓練，使模型越來越強大。那Life Long Learning看起來只是訓練更多的資料，會有什麼難點呢?

![Life Long Learning Example](/2021NTUML/Week17/example.JPG)
目前的Life Long Learning還沒達到先學習影音辨識，再學習影像辨識，接著翻譯任務這樣多重任務的水平，因此雖說是兩個任務，是指兩個類似的任務但資料Domain不同而已。首先將模型來學習任務一，完成後在拿相同的模型與參數來繼續學習任務二，但可以發現學習完任務二會忘記任務一。也許有人覺得模型太小，學習能力有限

![Life Long Learning Example 1](/2021NTUML/Week17/example-1.JPG)
但如果將任務一與任務二結合讓相同架構的模型學習，可以發現模型是有能力學習的。因此，模型要學會兩個任務需要一起學習才可以

![Life Long Learning Example 2](/2021NTUML/Week17/example-2.JPG)
接下來用QA任務來實驗看看，這裡使用簡單的QA任務，是早期Facebook提供的資料集，裡面包含了20個任務

![Life Long Learning Example 3](/2021NTUML/Week17/example-3.JPG)
上圖圖表橫軸是對應的任務，縱軸是在任務五的準確率。可以發現在任務五前準確率是0，因為還沒學習到任務五，當看過任務五時準確率100%，但當學習任務五後學習新任務準確率卻暴跌

![Life Long Learning Example 4](/2021NTUML/Week17/example-4.JPG)
有人可能懷疑模型本身沒有能力能多學這麼多任務，但如果將所有任務讓機器同時學可以發現模型是有能力的。這樣的情況被稱為`Catastrophic Forgetting`

![Wait a minute](/2021NTUML/Week17/wait.JPG)
有些人可能會想說那只要把多個任務組合起來就好了，為什麼要捨棄前任務讓機器學習新任務呢?使用Multi-task Training就能解決了吧?假設要學習第1000個任務又不忘記前面的任務的話，首先就須要有足夠的容量來儲存先前的任務，再來是一次學習1000個任務需要大量的時間來學習。在許多Life Long Learning研究中，會把Multi-task Training當成是上限，也就是先使用Multi-task Training當成上限，再使用Life Long Learning來逼近這界線

![Wait a minute 1](/2021NTUML/Week17/wait-1.JPG)
接下來可能有人會疑惑為什麼不把任務切開分別讓每個模型學習呢?為何要堅持一個模型學多個任務?使用每個模型學習單一任務確實不會有Catastrophic Forgetting的問題，但當任務量眾多時，也要要考慮是否有足夠的容量能儲存這些模型。Life Long Learning想了解的是，人類一個腦可以學習多件事，那要如何讓機器也能做到一樣的事

![Wait a minute 2](/2021NTUML/Week17/wait-2.JPG)
有些人可能會覺得這不就是Transfer Learning嗎?但其實關注點不同，Transfer Learning是學習好第一個任務，在第二任務表現如何，而Life Long Learning是第一個任務學完後再學第二個任務，第二個任務學完後在第一個任務表現如何

## Evaluation
![Evaluation](/2021NTUML/Week17/eval.JPG)
要評估Life Long Learning的模型好不好，首先需要有一系列的任務，通常會將同一個資料集透過一些規則來打亂來產生新任務，或者把類別分別拆出來成新任務

![Evaluation 1](/2021NTUML/Week17/eval-1.JPG)
假設有T個任務，首先會有T個隨機初始化的參數用來分別學習各個任務得到T個準確率，再來讓模型先學第一個任務，學完後接續學後續的任務並記錄各任務的準確率，以此類推到第T個任務的準確率都記錄下來後可以得到一個表格，這個表格可以用賴判斷Life Long Learning的模型好不好。如果看i > j的部分，表示看這模型是否忘記先前的任務，若看i < j的話，看模型在學到該任務前是否無思自通，學會之後的任務，也就是機器Transfer的能力。通常評估的方法是將最後一欄的準確率取平均，還有另一個評估方式是`Backward Transfer`，計算學完該任務以及學到最後時的準確率差多少，最後取平均，可以用來看模型遺忘的程度有多嚴重，因此這指標通常是負值

![Evaluation 2](/2021NTUML/Week17/eval-2.JPG)
還有`Forward Transfer`的方法，不過就不是用來評估Life Long Learning的模型，因為它是看在學到該任務前模型表現如何，因此是看該任務前在該任務的準確率

## Why Catastrophic Forgetting? 
![Why Catastrophic Forgetting](/2021NTUML/Week17/why-2.JPG)
接下來講解為什麼會發生Catastrophic Forgetting，假設有兩個任務，把它們對某兩個參數的Error Surface繪製出來，越藍色表示loss越小，因此在學習任務一時，會學到一組參數可以到任務一Error Surface的低點，將這組參數拿來繼續學習任務二，因為Error Surface不同了，所以這組參數的更新方向也會不同。可以發現學完任務二的參數可以達到任務二Error Surface的低點，所以在任務二表現好，但在任務一會變差，這就是Catastrophic Forgetting產生的原因。要解決此問題，就必須讓模型在學習新任務時，參數更新方向是往新任務與前任務共同的低點

## Selective Synaptic Plasticity
![Selective Synaptic Plasticity](/2021NTUML/Week17/select.JPG)
`Selective Synaptic Plasticity`方法的基本概念是，對於過去重要的參數，在學習新任務時盡量不要變，只改變不重要的參數就好。為了怕會有Catastrophic Forgetting產生，所以要更改Loss Function，新任務中更新的參數會減去過去任務學出來的參數，再乘上一個b值，這個值代表這個參數有多重要，我們有多希望它不被更新

![Selective Synaptic Plasticity 1](/2021NTUML/Week17/select-1.JPG)
當b值為0時，表示這個參數不被限制，它就會被更新，就有可能會產生Catastrophic Forgetting。那如果b值非常大，表示b值非重要不希望被更新，那有可能會有Intransigence的問題，也就是新任務會學不起來。那要怎麼設定b值呢?

![Selective Synaptic Plasticity 2](/2021NTUML/Week17/select-2.JPG)
為了要判斷這個參數是否重要，所以如上圖，把參數水平移動的話，發現loss變化不大，表示這個參數再這個任務不太重要，所以這個參數的b值會比較小。若把參數縱向移動，會發現loss變化很大，表示這個參數對這任務很重要，那麼這參數的b值就會比較大

![Selective Synaptic Plasticity 3](/2021NTUML/Week17/select-3.JPG)
這前面b值的設法，參數在橫向上限制小，縱向上限制大，所以參數更新的方向會在橫向上變化大，這樣更新後的參數在任務一與任務二都可以得到較小的loss，就比較不會有Catastrophic Forgetting的問題

![Selective Synaptic Plasticity 4](/2021NTUML/Week17/select-4.JPG)
上圖示文獻實驗的結果，橫軸為任務依序的訓練，縱軸為各任務分別的準確率。藍色線是一般的訓練，沒有使用Selective Synaptic Plasticity，也就是b值為0，可以發現隨著任務的學習，準確率一直在降低。綠色線是將每個參數的b值設為1，雖然在任務A訓練到任務C在任務A的準確率沒有問題，但在任務B與C的準確率會發生Intransigence，限制過大導致模型學不起新的任務。紅色線是有的參數b值大有的小，就可以發現不會有以上的問題

若想更了解b值設定與計算方法可以參考以下的連結 : 
* Overcoming catastrophic forgetting in neural networks : [Link](https://arxiv.org/abs/1612.00796)
* Continual Learning Through Synaptic Intelligence : [Link](https://arxiv.org/abs/1703.04200)
* Memory Aware Synapses: Learning what (not) to forget : [Link](https://arxiv.org/abs/1711.09601)
* Riemannian Walk for Incremental Learning: Understanding Forgetting and Intransigence : [Link](https://arxiv.org/abs/1801.10112)
* Sliced Cramer Synaptic Consolidation for Preserving Deeply Learned Representations : [Link](https://openreview.net/forum?id=BJge3TNKwH)

![Gradient Episodic Memory](/2021NTUML/Week17/gem.JPG)
早期有個方法叫`Gradient Episodic Memory (GEM)`，它不是在參數上做限制，而是在參數更新的方向做限制。當學習任務二要更新參數時，會參考任務一的更新方向來做修改，修改後的與任務二更新的方向做內積要大於0且越接近越好。但這方法會需要先前任務的資料，這樣才可以找到先前任務的更新方向，所以有點違反Life Long Learning的精神

## Additional Neural Resource Allocation
![Additional Neural Resource Allocation](/2021NTUML/Week17/progress.JPG)
另一個解決Catastrophic Forgetting的方法是`Additional Neural Resource Allocation`，使用這方法的模型是`Progressive Neural Networks`，想法是學習任務一時有一個模型，學任務二時多開一個模型但會使用任務一模型Hidden Layer的參數，學習任務三時的模型會使用任務一與任務二模型的參數。這是一個有效解決Catastrophic Forgetting的方法，但需要額外的空間來產生額外的Neuron，且增加的空間與任務量成正比

若想更了解Progressive Neural Networks可以參考以下連結 : 
* Progressive Neural Networks : [Link](https://arxiv.org/abs/1606.04671)

![Additional Neural Resource Allocation 1](/2021NTUML/Week17/packnet.JPG)
另一個方法是`PackNet`，先開一個大模型，當有任務進來時限定只能使用部分參數，這樣參數量不會隨的任務量增加，但這方法與Progressive Neural Networks很像，只是一開始先設定一個大模型而已

若想更了解PackNet可以參考以下連結 : 
* PackNet: Adding Multiple Tasks to a Single Network by Iterative Pruning : [Link](https://arxiv.org/abs/1711.05769)

![Additional Neural Resource Allocation 2](/2021NTUML/Week17/CPG.JPG)
Progressive Neural Networks與PackNet的想法可以結合起來變`Compacting, Picking, and Growing (CPG)`，既可以增加新參數，而每次保留部分參數做訓練

若想更了解CPG可以參考以下連結 : 
* Compacting, Picking and Growing for Unforgetting Continual Learning : [Link](https://arxiv.org/abs/1910.06562)

## Memory Reply
![Memory Reply](/2021NTUML/Week17/reply.JPG)
第三個解決Catastrophic Forgetting的方法是使用Generative model來產生先前任務的Pseudo Data，在訓練任務一時額外訓練一個模型能產生任務一的資料，學習任務二時，使用任務二的資料與Generative model產生的任務一資料一起訓練，這樣就不會忘記任務一，這個Generative model接下來就能產生任務一與任務二的資料供後續任務使用

若想更加了解這方法的應用可以參考以下連結 : 
* Continual Learning with Deep Generative Replay : [Link](https://arxiv.org/abs/1705.08690)
* FearNet: Brain-Inspired Model for Incremental Learning : [Link](https://arxiv.org/abs/1711.10563)
* LAMOL: LAnguage MOdeling for Lifelong Language Learning : [Link](https://arxiv.org/abs/1909.03329)

![Adding new classes](/2021NTUML/Week17/add.JPG)
先前方法的每個任務都有相同數量的class，若每個任務的class不同有辦法解嗎?答案是有的，若想解決這問題可以參考以下連結 : 
* Learning without Forgetting : [Link](https://arxiv.org/abs/1606.09282)
* iCaRL: Incremental Classifier and Representation Learning : [Link](https://arxiv.org/abs/1611.07725)

## Curriculum Learning
![Curriculum Learning](/2021NTUML/Week17/curriculum.JPG)
有許有些人有疑問若調換任務順序會怎樣?向上圖做手寫數字分類，先做複雜的任務一再做簡單的任務二與順序顛倒後的結果，因為有些順序有Catastrophic Forgetting，有些沒有。因此想了解什麼樣的順序對學習是有效的叫`Curriculum Learning`

想了解如何做Curriculum Learning可以參考以下連結 : 
* TASKONOMY : Disentangling Task Transfer Learning : [Link](http://taskonomy.stanford.edu/#abstract)

## Three scenarios for Life Long Learning
先前介紹的只是Life Long Learning中的一個情境，若想了解其他兩個關於Life Long Learning的情境可以參考以下連結 : 
* Three scenarios for continual learning : [Link](https://arxiv.org/abs/1904.07734)