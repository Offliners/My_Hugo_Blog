---
title: "NTU Machine Learning Week 16 - Reinforcement Learning"
date: 2021-06-11T17:14:52+08:00
draft: false
toc: true
comment: true
description: NTU Machine Learning 2021 Spring Week 16 - Reinforcement Learning

categories:
  - NTU 機器學習 note
tags:
  - Machine Learning
  - Deep Learning
  - NTU ML 2021
---

![cover](/2021NTUML/Week16/cover.JPG)

NTUML 2021 Spring Course Syllabus: [Link](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

Youtube Video Link (Chinese): [`Video 1`](https://youtu.be/XWukX-ayIrs) [`Video 2`](https://youtu.be/US8DFaAZcp4) [`Video 3`](https://youtu.be/kk6DqWreLeU) [`Video 4`](https://youtu.be/73YyF1gmIus) [`Video 5`](https://youtu.be/75rZwxKBAf0)

Youtube Video Link (English): [`Video 1`](https://www.youtube.com/watch?v=0oucZNfBXlI) [`Video 2`](https://youtu.be/jbN0oYLtXps) [`Video 3`](https://youtu.be/Cf-WkM-Xef0) [`Video 4`](https://youtu.be/pibO_5JhQ4U) [`Video 5`](https://youtu.be/9H3ShV57lHs)

## What is RL ?
![ML](/2021NTUML/Week16/ML.JPG)
機器學習最主要在做的事情就是找一個Function，Reinforcement Learning也是機器學習的一種，所以它也是在找一個Function。在RL中會有個Actor與Environment，首先Environment會給Actor一個Observation，也就是Actor的輸入，接著Actor會有輸出Action去影響Environment來產生新的Observation，互動期間Environment會給Actor一些Reward來告訴Actor這樣的Action是否是好的。所以Actor其實就是我們要找的Function，它要能讓Actor得到Reward的總和是最大的

## Example: Playing Video Game
![Game](/2021NTUML/Week16/game.JPG)
接下來使用Space invader來舉例，遊戲玩法是開火擊退所有外星人，而玩家有護盾可以抵擋外星人的攻擊(玩家的攻擊也會破壞護盾)，擊退外星人可以獲得分數。遊戲終止有兩個情況，一個是退所有外星人，另一個情況是玩家被外星人擊毀

![Game 1](/2021NTUML/Week16/game-1.JPG)
當要訓練RL來玩這遊戲的話，Actor就是玩家，Environment是遊戲本身，Observation是遊戲畫面，Action是輸出，有三種可能的行為分別為向左向右還有開火。如果Action選擇向右的話，reward是0因為沒有擊退外星人

![Game 2](/2021NTUML/Week16/game-2.JPG)
當遊戲畫面變化時，表示有新的Observation，這時就會有新的Action，如果選擇開火並擊退外星人的話，就會得到reward等於5。因此學習的目標是，讓Actor能得到的reward總和是最大的

## Example: Learning to play Go
![Alpha Go](/2021NTUML/Week16/go.JPG)
若是Alpha Go的話，Environment是人類對手，Observation就是棋盤，Action就是落子的位置，透過對手的落子來產生新的Observation，然後持續下去

![Alpha Go 1](/2021NTUML/Week16/go-1.JPG)
在下圍棋中，任何行為都沒辦法得到reward，因此會定義贏了才能得到reward為1，輸了得到-1分

## How to train RL ?
![train](/2021NTUML/Week16/train.JPG)
那要怎麼訓練RL呢?與先前的相同，第一步是有未知數的Functin，這些未知數是要被找出來的，第二步是定義loss function，第三步是最小化loss

![Step 1](/2021NTUML/Week16/step1.JPG)
首先是第一步，會使用Network，輸入是遊戲畫面，輸出是每個行為的分數，這件事情其實就是分類。Network架構可以自行設計，若輸入是遊戲畫面可能會使用CNN，若想看遊戲開始到現在發生的情況就有可能使用RNN或者Transformer。最後機器會採取什麼Action取決於輸出的分數，通常會轉換成機率並隨機Sample，那為什麼不直接選擇機率最高的呢?多數RL的應用中，會選擇隨機Sample，這樣子即使看到相同畫面也有可能採取不同行為，來增加隨機性

![Step 2](/2021NTUML/Week16/step2.JPG)
第二步要定義Loss，在Environment與Actor的互動中會得到許多reward直到遊戲結束，從遊戲開始到結束叫一個`Episode`，將這些reward總合起來叫Total reward或者Return，這是我們要去最大的目標，加上負號就是loss，也就是要最小化

![Step 3](/2021NTUML/Week16/step3.JPG)
最後是第三步，Environment會產生Observation叫s1，再來將s1輸入給Actor產生Action叫a1，持續下去到遊戲終止，這些s與a形成的Sequence叫做`Trajectory`。得到的Reward能當成是一個Function，有些遊戲只看Action就能決定，但通常會看當下的Observation，像Space invader需要觀察Observation是否擊退敵人才能判斷是否得到reward。但這不是一般的Optimization問題，首先是Actor的輸出是有隨機性，而且Environment是個黑盒子，只知道輸入後會產生回應，但對應關係我們不知道。因此在RL中解決Optimization問題使用的方法會與之前不同

## Policy Gradient
![Control](/2021NTUML/Week16/control.JPG)
首先思考要Actor在輸入Observation後如何輸出Action，先前提到這是個分類問題，當輸入某個Observation時希望能輸出向左，那麼則透過Cross-entropy來最小化loss就好，若希望不要向左則加負號即可，因此只要有適當的Label與loss，就可以控制Actor做我們想做的行為

![Control 1](/2021NTUML/Week16/control-1.JPG)
因此要Actor看到s時向左，看到s'時不要向左的話，只要定義loss是兩個case的Cross-entropy相減，再去最小化即可

![Control 2](/2021NTUML/Week16/control-2.JPG)
只要收集好Actor輸入s的對應Action，就可以定義好loss去最小化

![Control 3](/2021NTUML/Week16/control-3.JPG)
甚至可以給定行為一個分數，來代表我們期望Actor在特定Observation時輸出特定Action的程度。解決好loss，但問題是label和期望行為的A值怎麼來的呢?

![Version 0](/2021NTUML/Week16/v0.JPG)
首先介紹最簡單的版本，但其實不正確。這個方法是Actor與Environment互動時，將產生的Action都記錄下來，這些資料通常是蒐集多個episode產生的，記錄好後會去評價這些Action是好還是不好，那要如何評價呢?可以透過每個Action得到的reward來當評價的依據，如果reward是正的表示這是個好的Action；反之，負的就不是好的Action。但這不是個好方法來訓練Actor，因為會使Actor短視近利

![Version 0-1](/2021NTUML/Week16/v01.JPG)
由於Action 1會影響之後的行為，每個行為並不是獨立的，所以這方法只會選擇當下好的，不會考慮之後，因此需要`Reward Delay`來犧牲短期利益來獲取更長遠的reward。例如在Space invader中，使用最簡單的方法來評量Action的話，Actor就只會射擊，因為只有射擊會獲得Reward，左右移動並不會有，但不代表左右移動不重要，有時左右移動可以瞄準來獲得更多的Reward

![Version 1](/2021NTUML/Week16/v1.JPG)
這時就有改良的版本，評量Action的標準是這個Action之後所獲得的Reward總和。但仔細想想這版本也會有些問題，若遊戲進行很長，最後所獲得的reward是因為一開始的Action嗎?有可能是之間的Action所造成的

![Version 2](/2021NTUML/Week16/v2.JPG)
所以又有改良版本，乘上一個Discount Factor，所以離當下Action較近的reward有較大的權重，太遠影響較小的獲得到比較小的權重

![Version 3](/2021NTUML/Week16/v3.JPG)
更進階的改良版本，將評價的分數做進一步的區分好壞，因為好壞是相對的，若大家的評價分數都是正的就難以區分，簡單的區分是設定一個baseline，高於這個標準就是正的，反之是負的，那baseline要怎麼定義呢?接下來版本會提到

![Policy Gradient](/2021NTUML/Week16/policy.JPG)
實際執行蒐集資料並評價Action的步驟如上圖，最後就是計算loss來更新參數，但有個神奇的地方是蒐集資料在迴圈中，一般蒐集資料是在迴圈外但RL居然是在迴圈內

![Policy Gradient 1](/2021NTUML/Week16/policy-1.JPG)
因此當蒐集完資料後就會去更新參數，但只會更新一次，更新完就要再蒐集新的資料，這就是為什麼訓練RL很花時間，假設要更新400次參數，那資料也要蒐集400次。那為什麼更新完參數要重新蒐集資料呢?

![Policy Gradient 2](/2021NTUML/Week16/policy-2.JPG)
因為更新完參數後Actor變強了可能就不適用在舊資料，因此要用變強的Actor來蒐集資料再更新參數

若想了解更多Policy Gradient可以觀看此影片 : 
* Deep Reinforcement Learning : [Video](https://youtu.be/W8XF3ME8G2I)

![On-policy v.s. Off-policy](/2021NTUML/Week16/on-off.JPG)
當訓練的Actor和與Environment互動的Actor是相同的話就是`On-policy`，像前面提到的Actor就是。若訓練的Actor跟與Environment互動的Actor是不同的話就是`Off-policy`，這樣的Actor訓練可以使用先前跟新的參數，所以可以不用一直蒐集資料。最有名的Off-policy方法是PPO(Proximal Policy Optimization)，有興趣的人可以參考以下影片 : 
* Proximal Policy Optimization (PPO) : [Video](https://youtu.be/OAKAZhFmYoI)

![Exploration](/2021NTUML/Week16/exp.JPG)
RL在蒐集資料有個重要的技術是`Exploration`，也就是在Actor中加入隨機性，讓Actor在與Environment互動時能有多個情況，讓Actor學習在各種情況採取各種Action的好壞。因此有些人會刻意加大Entropy或者在參數中加入雜訊，以此提高Actor採取機率較低的Action

## Actor-Critic
![Critic](/2021NTUML/Week16/critic.JPG)
`Critic`是用來評估Actor的好壞，有些透過觀察Observation，有些則還會多觀察Action。有Critic後接下來是`Value Function`，輸入是Observation，輸出是一個數值，這個數值是Value Function看到這個Observation後預測接下來獲得的Discounted Cumulated Reward是多少

![Critic 1](/2021NTUML/Week16/critic-1.JPG)
接下來介紹怎麼訓練Critic，有兩種常用的方法，第一種是`Monte-Carlo (MC) based approach`，直接拿Actor與環境互動很多輪，接下來輸入互動時看到的Observation得到預估的分數，和實際得到的分越接近越好

![Critic 2](/2021NTUML/Week16/critic-2.JPG)
另一種做法是`Temporal-difference (TD) approach`，不需要玩完整場遊戲，只需要看當下的Observation、Action、Reward與下一個Observation就可以來訓練，適合用於長度較長的遊戲或者沒有結局的遊戲。透過上面的式子，可以發現雖然我們不知道當下的Observation實際得到多少分，但可以知道當下Observation與下一個Observation之間的差值

![vs](/2021NTUML/Week16/vs.JPG)
假設有一個簡單的遊戲，第一回合是看到Sa得到reward為0，接下來看到Sb得到reward為1就結束，接下來數回合只看到Sb得到reward後就結束。透過Value Function來計算Sb期望獲得的分數，8場有6場得到1，2場得到0，因此為3/4，那Sa為多少呢?有可能是0或者3/4，0的話是MC的想法，3/4是TD的想法，兩者皆是對的，只是背後的假設不同

![Version 3.5](/2021NTUML/Week16/v35.JPG)
將先前提到的Version 3改良後，將Baseline變成當下Observation透過Value Function得到的分數，那為什麼可以這麼做呢?

![Version 3.5-1](/2021NTUML/Week16/v35-1.JPG)
輸入當下的Observation，因為在Actor會隨機Sample出Action來增加隨機性，因此同樣的Observation會有多種可能Action，將這些可能透過Value Function來計算出數值。而當執行某一個Action得到的值大於Value Function來計算出數值，表示這個Action比Sample多次Action的平均還要好；反之，還要差。但可能會覺得疑惑的點是，這個Action也是Sample出來的，拿一個隨機的Action去減掉平均會比較好嗎?

![Version 4](/2021NTUML/Week16/v4.JPG)
所以就有了改良版本，拿平均減去平均，將執行Action後的Observation輸入Value Function來計算平均，這樣子就可以知道採取這個Action的期望Reward與隨機Sample的期望Reward哪個比較好，這就是有名的`Advantage Actor-Critic`

![Tip](/2021NTUML/Week16/tip.JPG)
訓練Actor-Critic有個小技巧，在實作時，由於Actor與Critic都是個Network，因為輸入都是Observation輸出不同而已，所以可以共用Network，如果輸出不同的Action就是Actor，輸出數值就是Critic

![DQN](/2021NTUML/Week16/dqn.JPG)
在Reinforcement Learning中還有個做法，直接採取Critic來決定Action，最知名的就是`Deep Q Network (DQN)`，若想了解DQN可以觀看以下影片與論文 : 

* Q-learning (Basic Idea) : [Video](https://youtu.be/o_g9JUMw1Oc)
* Q-learning (Advanced Tips) : [Video](https://youtu.be/2-zGCx4iv_k)
* Rainbow: Combining Improvements in Deep Reinforcement Learning : [Link](https://arxiv.org/abs/1710.02298)

## Reward Shaping
![Sparse Reward](/2021NTUML/Week16/sparse.JPG)
到目前為止是拿Actor與Environment互動，再把得到的Reward做一些處理來訓練Actor，但如果Reward幾乎都是0要怎麼辦?Reward幾乎都是0表示不管執行什麼Action都沒差，這樣會難以訓練Actor，因此需要想辦法提供額外的Reward來幫助Agent學習，這就是`Reward Shaping`。下面影片示範一款遊戲叫VizDoom。這款遊戲中殺敵人可以加分，被殺掉會扣分，單使用遊戲中的reward來訓練Agent會很難訓練起來

{{< youtube 94EPSjQH38Y>}}

![VizDoom](/2021NTUML/Week16/vizdoom.JPG)
這遊戲中第一名的隊伍定義的Reward Shaping如上表所示，像第二點被扣血的話扣0.05分，如果沒有Reward Shaping的話需要等機器被殺死後才有可能學到這是不好的事情，有許多事情是在遊戲中不會得到分數，需要人類強加讓機器來學習。還有些比較有趣的reward，像第七點和第八點，如果一直待在原地就扣0.03分，動一下給很小的分數，因為一開始機器不強，出去有高機率就被敵人打死，所以沒有這樣的機制可能會讓機器一直待在原地。又為了怕機器可能轉圈或者一直躲敵人，所以有第一點活著就扣0.008分，強迫機器出去與敵人對戰。看到這些Reward Shaping，可以得知這需要對Environment有一定程度的了解

## No Reward: Learning from Demonstration
![No reward](/2021NTUML/Week16/noreward.JPG)
對於像遊戲之類的事情，很容易定義出Reward，但如果reward都沒有呢?或許可以使用Reward Shaping，但有時機器會學出令人意想不到的事，有個方法可以解決叫`Imitation Learning`，在這方法中Actor仍然會與Environment互動，但不會得到任何reward，不過會有Expert與環境互動，Expert通常是人類，將互動記錄下來給Actor學習

![Supervised Learning](/2021NTUML/Week16/supervised.JPG)
或許有人會覺得這不就是`Supervised Learning`嗎?沒錯，因為機器只會去模仿Expert的行為叫做`Behavior Cloning`，但如果機器只會複製Expert行為會有些問題，例如自駕車，當Expert都順利過彎時，機器也學會過彎，但這樣機器在遇到要撞牆的情況時就會不知道怎麼辦，因為Expert沒有示範過

![Inverse Reinforcement Learning](/2021NTUML/Week16/inverse.JPG)
因此就有更進階的方法叫`Inverse Reinforcement Learning`，透過Expert示範的紀錄與Environment，去學習說Reward該怎麼定義，當有Reward Function就可以用之前介紹的Reinforcement Learning來訓練

![Inverse Reinforcement Learning 1](/2021NTUML/Week16/inverse-1.JPG)
在這方法的基本原則是Expert的行為會被定義為獲得最高的Reward，Actor會與Environment互動來獲得紀錄，接著定義一個Reward Function，這Function會將Expert的行為評的比較高分，Actor會比較低分，再來Actor就會去更新參數去最大化Reward Function，最終學到的Reward Function就是我們要的

![Inverse Reinforcement Learning 2](/2021NTUML/Week16/IRL.JPG)
上圖為Inverse Reinforcement Learning的流程，整個框架就像是GAN

![Inverse Reinforcement Learning vs GAN](/2021NTUML/Week16/GAN.JPG)
兩者有異曲同工之妙，只是不同的名字而已。將Inverse Reinforcement Learning套用在機器手臂上如下影片示範

{{< youtube hXxaepw0zAw>}}

![Reinforcement learning with Imagined Goals](/2021NTUML/Week16/RIG.JPG)
想要訓練機器手臂，除了人類示範以外，甚至還能給機器圖片，要它達到圖片的結果，若想了解更多可以參考以下連結 : 

* Visual Reinforcement Learning with Imagined Goals : [Link](https://arxiv.org/abs/1807.04742)
* Skew-Fit: State-Covering Self-Supervised Reinforcement Learning : [Link](https://arxiv.org/abs/1903.03698)