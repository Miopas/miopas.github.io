---
layout: post
title: IBM Watson 相关 paper 阅读笔记（二）
categories: [问答系统]
description: IBM Watson 相关 paper 阅读笔记
keywords: 问答系统, chatbot, QA system
---

更新四篇。

本篇涉及：
6. Typing Candidate Answers using Type Coercion
7. Textual Evidence Gathering and Analysis
8. Relation Extraction and Scoring in DeepQA
9. Structured Data and Inference in DeepQA

---
#### 6. Typing Candidate Answers using Type Coercion
&emsp;&emsp;Hypothesis Generation 是[ DeepQA 框架](https://github.com/Miopas/miopas.github.io/blob/master/_posts/deepqa_architecture.png)中的一个主要模块，其功能是产生 candidate answers 及对应的一系列 feature。这篇论文是关于 Hypothesis Generation 中的一个子组件 Type Coercion（TyCor），TyCor 模块的功能是给 candidate answers 计算一个 type，以及给出相应的 type score，作为 candidate answer 的一个 feature。

&emsp;&emsp;传统的 QA system 的做法是，先确定 answer 的 type，然后从对应的 type 中查找可能的 candidate answers（type-and-generate approach）。但是 Watson 的做法是相反的，先不考虑 type，产生 candidate answers，再用其他的方法去获取每一个候选的 type，然后判断这个 type 和 Question Analysis 阶段所获得的的 LAT 的 type 是否符合。

略读。

---
#### 7. Textual Evidence Gathering and Analysis
&emsp;&emsp;在 DeepQA 框架中，得到 candidate answers 和对应的 answer score 之后，接下来要对每一个 candidate answer 计算 evidence score。概括地说，evidence score 体现了一个 candidate answer 的可信度。
这篇论文介绍了其中一种 evidence score——passage score 是如何计算的。

&emsp;&emsp;内容主要分为两部分： 
* 如何检索到这样的 passage：既包含了 candidate answer 又和 question 相关。
* 对于检索到的 passage，如何根据 question 和 passage 的相关程度来计算 passage socre。

---
###### 7.1）检索 passage
&emsp;&emsp;这里提出了一种新方法：`Surpporting Evidence Retrival (SER)`。其核心思想是把 candidate answer 作为 search query 的一部分来检索。这样做能召回更多的 passage。

###### 7.2）计算 passage score
&emsp;&emsp;在计算 passage score 之前，需要做一个预处理工作：对于 question 和 passage 建立一个 `Syntactic-semantic graph`。这个 graph 是在 PAS（见 [Deep Parsing in Watson](https://github.com/Miopas/miopas.github.io/blob/master/_posts/2017-12-26-watson-paper-02.md)） 的基础上进一步衍生得到的。

&emsp;&emsp;Syntactic-semantic graph 和 PAS 一样，是一个包含了实体信息和实体之间的关系的一个数据结构。其 node 包含了实体信息，edge 包含了实体之间的句法关系或者语义关系。

&emsp;&emsp;其后介绍了计算 passage score 的 4 种 algorithm，不展开。

---
#### 8. Relation Extraction and Scoring in DeepQA
&emsp;&emsp;在 [Deep Parsing in Watson](https://github.com/Miopas/miopas.github.io/blob/master/_posts/2017-12-26-watson-paper-02.md) 中提到的 PAS 和 ESG，简单介绍了句法层面和语义层面的关系抽取。这篇论文进一步介绍了语义层面的关系抽取。

&emsp;&emsp;文章介绍了 rule-based 和 statistical 的两种方法。统计的方法主要是 `特征工程` + `svm 分类器`。略读。

---
#### 9. Structured Data and Inference in DeepQA
&emsp;&emsp;这篇是关于在 DeepQA 中 structured data 的获取与使用。前面几篇论文介绍 DeepQA 的一些组件的时候，都提及了如何利用 structured data，这篇 paper 则是进一步描述了以下几个方面：
* 为什么要使用 structured data
* 如何建立 structured data 的数据库
* 在 Watson 中如何利用 structured data

略读。
