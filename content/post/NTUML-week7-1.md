---
title: "NTU - 機器學習 Week 7 - Transformer"
date: 2021-04-13T20:43:16+08:00
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

![cover 1](/2021NTUML/Week7/cover-1.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://www.youtube.com/watch?v=n9TlOhRjYoc) [`Video 2`](https://youtu.be/N6aRv06iv2g)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=zmOuJkH9l9M) [`Video 2`](https://www.youtube.com/watch?v=fPTj5Zh1ACo)

## Applications of Sequence-to-sequence (Seq2seq)
![Sequence-to-sequence](/2021NTUML/Week7/seq2seq.JPG)
當模型輸入是sequence時，輸出可以是與輸入相同長度的sequence，也可能是未知長度的sequence，由機器去決定。輸出是未知長度sequence的例子如
* 語音辨識 Speech Recognition
* 機器翻譯 Machine Translation
* 語音翻譯 Speech Translation

這時可能有人問為何要做語音翻譯?直接語音辨識再機器翻譯不就好了?因為世界上有眾多語言，有些語言甚至連文字都沒有，沒有文字的語言難以做語音辨識

若想了解台語語音辨識可點以下連結:
To learn more : [Link](https://sites.google.com/speech.ntut.edu.tw/fsw/home/challenge-2020)

輸入聲音，輸出辨識後的文字是`語音辨識 Speech Recognition`。反過來，輸入文字，輸出聲音訊號，就是`語音合成 Text-to-Speech Synthesis`

### Seq2seq for Chatbot
![Seq2seq for Chatbot](/2021NTUML/Week7/chatbot.JPG)
文字方面，也可透過使用Seq2seq的模型來訓練，例如可以訓練聊天機器人，輸入輸出都是文字，訓練資料是一堆對話的內容

### Most Natural Language Processing applications
![Most Natural Language Processing applications](/2021NTUML/Week7/NLP.JPG)
很多NLP的問題可以看成是QA的問題，多數QA的問題又可以透過Seq2seq解決。輸入問題與文章，輸出就是問題的答案

想了解更多NLP的處理，可參考以下連結
To learn more : 
* The Natural Language Decathlon: Multitask Learning as Question Answering : [Link](https://arxiv.org/abs/1806.08730)
* LAMOL: LAnguage MOdeling for Lifelong Language Learning : [Link](https://arxiv.org/abs/1909.03329)
* DEEP LEARNING FOR HUMAN LANGUAGE PROCESSING 2020 SPRING : [Link](https://speech.ee.ntu.edu.tw/~hylee/dlhlp/2020-spring.html)

### Seq2seq for Syntactic Parsing
![Seq2seq for Syntactic Parsing](/2021NTUML/Week7/syntacticparsing-1.JPG)
Seq2seq也可用於文法剖析，輸入是一段文字，但輸出就不是sequence，而是樹狀結構用來分析各文字的組成

![Seq2seq for Syntactic Parsing](/2021NTUML/Week7/syntacticparsing-2.JPG)
不過樹狀結構也可以看成是一段sequence，這樣就可以套用Seq2seq來處理

To learn more : 
* Grammar as a Foreign Language : [Link](https://arxiv.org/abs/1412.7449)

### Seq2seq for Multi-label Classification
![Seq2seq for Multi-label Classification](/2021NTUML/Week7/multilabel.JPG)
首先要區分`Multi-label Classification` 與`Multi-class Classification`。Multi-class Classification是多個class由機器去選擇一個，Multi-label Classification是指同一個東西但屬於可能有多個class。那要怎麼處理Multi-label Classification的問題呢?可以透過Seq2seq，比如輸入文章，輸出是他所屬於的class

To learn more : 
* Order-free Learning Alleviating Exposure Bias in Multi-label Classification : [Link](https://arxiv.org/abs/1909.03434)
* Order-Free RNN with Visual Attention for Multi-Label Classification : [Link](https://arxiv.org/abs/1707.05495)

### Seq2seq for Object Detection
![Seq2seq for Object Detection](/2021NTUML/Week7/object.JPG)
甚至連物件偵測也能透過Seq2seq來解決

To learn more : 
* End-to-End Object Detection with Transformers : [Link](https://arxiv.org/abs/2005.12872)

## Seq2seq
![Seq2seq](/2021NTUML/Week7/seq2seq-1.JPG)
一般的Seq2seq會由encoder與decoder組成，上圖是Seq2seq早期的架構

To learn more : 
* Sequence to Sequence Learning with Neural Networks : [Link](https://arxiv.org/abs/1409.3215)

![Transformer](/2021NTUML/Week7/transformer.JPG)
現今主流架構會是`Transformer`

## Encoder
![Encoder](/2021NTUML/Week7/encoder.JPG)
Encoder在做的事情是輸入一排向量，輸出也是一排向量。Transformer中的encoder就是self-attention，透過下圖來簡單介紹

![Transformer Emcoder 1](/2021NTUML/Week7/transformer-1.JPG)
上圖每個block都是輸入一排向量，輸出一排向量。而block中會先對輸入做self-attention，接著送入fully connected network中，最後就是輸出的向量

![Transformer Emcoder 2](/2021NTUML/Week7/transformer-2.JPG)
而實際的Transformer中，會把self-attention輸出的向量加上原本的輸入，這樣的行為叫做`Residual`。接著會把residual後的向量做`Layer Normalization`，不同於Batch Normalization的是Batch Normalization是對同一個維度不同的feature做計算，但Layer Normalization不同的維度同一個的feature做計算。做完Layer Normalization得到的輸出就會是fully connected network的輸入，這裡也會套用residual與Layer Normalization的架構

![Transformer Emcoder 3](/2021NTUML/Week7/transformer-3.JPG)
上述介紹的簡易模型也就是Transformer encoder中block處理的方式，而這複雜的block再之後會介紹的BERT也會用到

若想了解更多為何要這樣encoder，可參考以下連結
To learn more : 
* On Layer Normalization in the Transformer Architecture : [Link](https://arxiv.org/abs/2002.04745)
* PowerNorm: Rethinking Batch Normalization in Transformers : [Link](https://arxiv.org/abs/2003.07845)

## Decoder
![Decoder](/2021NTUML/Week7/decoder.JPG)
Decoder有分兩種:
* Autoregressive (AT) 
* Non-autoregressive (NAT)

### Autoregressive
![Autoregressive](/2021NTUML/Week7/autoregressive.JPG)
如果是語音辨識，那會將聲音訊號輸入至encoder，輸出會是一排向量，接著這些向量會輸入到deocder來產生語音辨識的結果。那decoder是怎麼處理的呢?首先要有個特殊符號begin，通常是個one-hot vector，然後decoder會輸出一個向量，這個向量長度是自行定義的，假設是做中文的語音辨識，那向量長度可能就會是中文方塊字的數目，由於這向量通常會經過softmax產生，所以會把向量中分數最高的當成輸出，會由one-hot vector表示

![Autoregressive 1](/2021NTUML/Week7/autoregressive-1.JPG)
接著將剛剛輸出的one-hot vector與begin當成decoder的輸入，就會輸出向量中得分最高的值再用one-hot vector，接著再將begin與兩個one-hot vector輸入至decoder，又得到一個新的one-hot vector，以此類推。簡言之，decoder會將自己的輸出當成下一個自己的輸入，因此decoder辨識錯誤時，就可能會輸入錯的東西，那這樣會不會造成Error Propagation的問題?有可能，等等會說解決方法

![Decoder 1](/2021NTUML/Week7/decoder-1.JPG)
上圖為deocder在Transformer中的結構

![Encoder v.s Decoder](/2021NTUML/Week7/vs.JPG)
比較Transformer中的encoder與decoder，可以發現如果將decoder的紅色區域拿掉，兩者無太大差異，然後decoder會再經過softmax來產生機率。但可以發現decoder不同於encoder，是Multi-head Attention變成`Masked Multi-head Attention`

![Self-attention](/2021NTUML/Week7/self.JPG)
原本的Self-attention每個輸出需要看過每一個輸入才做決定

![Masked Self-attention](/2021NTUML/Week7/masked-self.JPG)
但Masked Self-attention變成輸出只看輸入左側的向量來決定，不看右側，所以只有最後一個輸出會考慮整個輸入才決定

![Self-attention 1](/2021NTUML/Week7/self-1.JPG)
可以比較兩者的計算，上圖是Self-attention的計算方式，若要計算出b2，q2需要拿a1~a4的key

![Masked Self-attention 1](/2021NTUML/Week7/masked-self-1.JPG)
如果是Masked Self-attention，若要計算出b2，q2只能拿a1、a2的key。那為何要不用Self-attention而是Masked Self-attention呢?

其實很直覺，思考剛剛提到的decoder的運作方式是逐一產生的，輸入的資料只有自己以前的，因此沒辦法參考全部的資訊

![Decoder 2](/2021NTUML/Week7/decoder-2.JPG)
但decoder還有個關鍵的問題，就是decoder需要決定輸出sequence的長度

![Decoder 3](/2021NTUML/Week7/decoder-3.JPG)
為了能讓decoder能停下來，需要一個特殊符號

![Decoder 4](/2021NTUML/Week7/decoder-4.JPG)
因此預期當機器讀到最後一個輸入時，能讀到這個特殊符號來停止輸出

### Non-autoregressive
![Non-autoregressive](/2021NTUML/Week7/NAT.JPG)
autoregressive是輸入begin，接著輸出w1，再把begin與w1當成輸入，直到輸出到end為止。Non-autoregressive不是一次產生一個詞，而是一次產生整個句子，因此一次輸入多個begin來產生整個句子。那這樣NAT要輸入多少begin來決定句子的長度?有兩種做法，第一種是定義另一個模型來預測句子輸出的長度，這個模型的輸出會是一個數值。另一種方法是直接輸入自行定義的句子長度上限，接著當輸出有end時，右邊的部分就都捨棄。

那NAT相較於AT有什麼好處呢?
* 平行化，句子能一次產生
* 能控制輸出句子的長度

雖然NAT有很多優勢，但目前NAT的表現是差於AT，由於NAT有著`Multi-modality`的問題，若想瞭解更多關於NAT的資訊可以參考以下連結:
* Non-Autoregressive Sequence Generation : [Link](https://youtu.be/jvyKmU4OM3c)

## Encoder - Decoder
![Encoder - Decoder](/2021NTUML/Week7/encoder-decoder.JPG)
那encoder與decoder怎傳遞資訊?接著要來講解剛剛Transform中紅色的區域`Cross Attention`，上圖可以發現Cross attention有兩個輸入來自encoder，一個來自decoder

![Cross attention](/2021NTUML/Week7/crossattention.JPG)
Cross attention是將由encoder產生k、v，由decoder產生q，來產生decoder接下來fully connected network輸入的資訊

![Cross attention 1](/2021NTUML/Week7/crossattention-1.JPG)
decoder接下來的輸入也是經過相同的Cross attention計算

![Cross attention 2](/2021NTUML/Week7/crossattention-2.JPG)
上圖灰階的部分是Cross attention的得分，此實驗來自以下論文
* Listen, attend and spell: A neural network for large vocabulary conversational speech recognition : [Link](Listen, attend and spell: A neural network for large vocabulary conversational speech recognition)

![Cross attention 3](/2021NTUML/Week7/crossattention-3.JPG)
但encoder與decoder有很多層，decoder只能拿encoder最後一層的輸出嗎?原始論文的實作是這麼做，但實際上可以有不同的連接方式

原始論文的連結:
* Layer-Wise Cross-View Decoding for Sequence-to-Sequence Learning : [Link](https://arxiv.org/abs/2005.08081)

## Training
![Training](/2021NTUML/Week7/training.JPG)
首先透過人工Label的方式，將聽到的聲音訊號標記Label，這些Label也是希望機器能產生的，機器會透過decoder的輸出，來產生分布的機率並選取機率最高的字當成輸出，因此這也是個分類的問題

![Training 1](/2021NTUML/Week7/training-1.JPG)
值得注意的是，decoder的輸入其實就是正確答案，這叫做`Teacher Forcing`，訓練時會輸入正確答案，但測試時沒有，因為deocder會看到自己的輸入，那這之間會有個mismatch，等等會介紹解決方式

## Tips of Training Seq2seq
### Copy Mechanism
![Copy Mechanism](/2021NTUML/Week7/copy.JPG)
剛剛的介紹都是deocder自己產生輸出，但對很多任務而言，decoder沒必要自己產生輸出，而是從輸入複製一些東西出來。這種行為可以用Chatbot，例如對於名稱或者對於機器不懂的話，機器不需要再重頭創造，只需要從輸入去複製作為輸出

![Copy Mechanism 1](/2021NTUML/Week7/summary.JPG)
或者是做摘要的時候，可以透過輸入文章給機器，機器會輸出這個文章的摘要。摘要中有許多的詞彙是直接從原文章中複製出來的，若想了解Seq2seq如何做到可以參考以下連結

* Pointer Network : [Video](https://youtu.be/VdOyqNQ9aww)
* Incorporating Copying Mechanism in Sequence-to-Sequence Learning : [Link](https://arxiv.org/abs/1603.06393)

### Guild Attention
![Guild Attention](/2021NTUML/Week7/guide.JPG)
有時候訓練Seq2seq的模型，會發現機器會漏字或漏看一些資訊，因此可以透過`Guild Attention`強迫機器一定要把每個東西都看過，Guild Attention會要求機器在做Attention時有固定的方式。假設做語音合成時，由於聲音訊號是由左到右，因此做Attention時應該也要由左到右，但如果發現attention是隨機的，那可能會出現一些錯誤

### Beam Search
![Beam Search](/2021NTUML/Week7/beam.JPG)
假設decoder只能產生兩個字A、B，decoder會選擇分數最高的，再把分數高的當成decoder的輸入再選擇分數高的當成輸出，這樣的流程叫`Greedy Decoding`。那這是最好的方法嗎?有沒有可能第一步雖然選擇低的，但第二步的得分更高?

若要找到最佳路徑要怎找?窮舉法太多可能性會行不通，因此可以透過`Beam Search`使用較有效的方法尋找可能解

![Beam Search 1](/2021NTUML/Week7/beam-1.JPG)
那Beam Search是否有用呢?有個論文是做Sentence Completion，機器讀入一段句子並由機器完成後半段，使用Beam Search時會發現機器不斷講重複的話(左)。若用其他方法，加入一些隨機性可發現結果雖然沒有很好，但會輸出一些較正常的句子(右)。因此對decoder而言，不見得分數最高的路是最好的，因此這要取決於任務本身的特性。假設任務答案非常明確，Beam Search會比較有幫助。但如果需要機器有些創造力時，Beam Search就表現比較差，這種問題往往需要在decoder加入隨機性，這非常神奇因為在decoder加入noise違背正常ML會做的想法，訓練時加入雜訊使機器看更多可能性是好的，但在測試時加入雜訊使測試狀況更複雜會使表現變差，但如Sentence Completion、TTS(Text-to-speech)等問題測試時加入隨機性反而表現比較好

### Optimizing Evaluation Metrics
![Optimizing Evaluation Metrics](/2021NTUML/Week7/BLEU.JPG)
在評估deocder時常用`BLEU score`，計算方式是將decoder輸出的整個句子與正確答案做比較去計算出BLEU score。但在訓練時不是這樣，訓練時是將詞彙分開考慮去minimize cross-entropy，由於訓練與測試的評分標準不同，因此minimize cross-entropy不見得能minimize BLEU score。那能在訓練時使用BLEU score?不能，因為BLEU score無法微分，如果把BLEU score當成loss無法算gradient

若在Optimization無法解決的問題，可以透過RL(Reinforcement Learning)來解決，可以參考以下連結
* Sequence Level Training with Recurrent Neural Networks : [Link](https://arxiv.org/abs/1511.06732)

![Exposure Bias](/2021NTUML/Week7/mismatch.JPG)
那訓練與測試不一致，測試時decoder看到自己的輸出，因此有可能看到錯誤的東西，但訓練時decoder是看到正確答案，這個不一致的現象叫`Exposure Bias`，那要怎麼解決呢?可以在訓練時給decoder看一些錯誤的東西，不要一直給decoder看正確答案

![Scheduled Sampling](/2021NTUML/Week7/schedule.JPG)
而這種方法叫`Scheduled Sampling`，但這會傷害到Transformer平行化的能力，因此不同的Seq2seq模型有不同Scheduled Sampling的方法，想了解更多可參考以下連結:

* Scheduled Sampling for Sequence Prediction with Recurrent Neural Networks : [Link](https://arxiv.org/abs/1506.03099)
* Scheduled Sampling for Transformers : [Link](https://arxiv.org/abs/1906.07651)
* Parallel Scheduled Sampling : [Link](https://arxiv.org/abs/1906.04331)