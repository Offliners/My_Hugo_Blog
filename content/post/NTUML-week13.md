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

Youtube Video Link (Chinese): [`Video`](https://youtu.be/WQY85vaQfTI)

Youtube Video Link (English): [`Video`]

## Why we need Explainable ML?
![Why we need Explainable ML?](/2021NTUML/Week13/issue.JPG)
模型得到正確答案不代表模型很聰明，因此在將來，模型得到正確答案時需要解釋為什麼得到這個答案。以上為現今機器學習應用的舉例，如銀行借貸款、醫療診斷、法律判決與自駕車等，模型需要為得出的答案給出合理的解釋才能使人信服

## Interpretable v.s. Powerful
![Interpretable v.s. Powerful](/2021NTUML/Week13/vs.JPG)
若採用較好解釋的模型如Linear model，但往往表現較差;若使用Deep model較難解釋，但表現較好。因此有人認為不應該使用Deep model，因為它是個黑盒子，但這有點削足適履，應該想辦法去提升模型的解釋力

![Decision tree](/2021NTUML/Week13/tree.JPG)
可能有人會提那使用`Decision tree`? Decision tree相較Linear model強大，也比Deep model好解釋，透過節點問題分析來選擇最終決斷

![Decision tree 1](/2021NTUML/Week13/tree-1.JPG)
但一棵Decision tree也可以有複雜的結構，而且實際上也不會只使用一棵，而是`Random forest`的技術，使Decision tree變得複雜難以解釋

## Goal of Explainable ML
![Goal of Explainable ML](/2021NTUML/Week13/goal.JPG)
