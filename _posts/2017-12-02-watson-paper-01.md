---
layout: post
title: IBM Watson 相关 paper 阅读笔记（一）
categories: [问答系统]
description: IBM Watson 相关 paper 阅读笔记
keywords: 问答系统, chatbot, QA system
---

这里是关于 IBM Watson 在  IEEE 发表的 17 篇关于开放领域的问答系统的 paper 的阅读笔记。

目录可以看[这里](https://gist.github.com/Miopas/19d6d44b6c21b6b2ba868b13c30fb892)。第一篇 *Introduction to “This is Watson”* 是一篇综述，介绍了整体框架，在此不讨论。

本篇涉及：
* Question Analysis: How Watson Reads a Clue
* Textual resource acquisition and engineering
* Automatic knowledge extraction from documents
* Finding needles in the haystack: Search and candidate generation

------
#### 1. Question Analysis: How Watson Reads a Clue

这篇介绍了 Watson 是怎么做 question analysis 的。主要内容是如何分解 question 和具体的抽取信息的方法。

###### 1.1）对于 question 定义了四个 critical elements：

* focus：question 中出现的和答案相关的片段。例如“he is...”中的 "he"；
* LATs(lexical answer types)：answer 的类型。是从 question 中抽取的暗示了 answer type 的 term。例如“president”；一般依赖于 focus；
* QClass：question 的类型，不一定出现在 question 的描述中；
* QSecton：question 中的一些特殊片段，和对 answer 的要求有关，例如"4-letter"。

###### 1.2）计算上述四种 elements 的方法：rule-based + statistics

其中提及的 Prolog 是一种 rule matching 的框架（待确认）。

这部分内容重点是计算 focus 和 LAT 的部分。LAT 对整体的效果影响比较大，描述的篇幅也比较长。

###### 1.3）总结

整体看来，在 Question Analysis 的过程中，rule 所占的比重还是挺大的。

最后在 Related work 这一节里提到了为什么要这样去分解 question。这些细节值得一读。

---
#### 2. Textual resource acquisition and engineering

这篇是关于如何利用 web data 作为 textual resource。

大数据量带来的两个问题：硬件的计算能力；如何获得高质量的数据。本文主要着眼于第二个问题，介绍了处理数据的三个过程。

ps. 这篇只是简单地看了一下。

处理数据的三个步骤：source acquisition，source transformation，source expansion

###### 2.1）source acquisition：从海量数据中获取和当前的任务相关的数据（对于 Watson 来说就是获取和 Jeopardy! 相关的 question 的数据）

这部分有一个细节。因为 Jeopardy! 这个游戏的问题涵盖内容很广，所以定义 domain 很困难。但是研究者观察到 Jeopardy! 这个游戏的问题都是对于观众来说相对感兴趣的问题，于是选择了 wikipedia 这个数据来源。因为 wikipedia 的文档就是该条目的爱好者们编辑的。

###### 2.2）source transformation：让系统更好地利用获得的数据

观察获得的数据会发现，all the source are not created euqal。
不同的来源的数据，title 和 document 的中有效信息的种类不同。
一类来源（title-oriented document）的文档，title 通常是概念或者实体，而 document 中是关于这个概念或者实体的信息，如 wikipedia；
另一类文档，如新闻文档，title 通常描述了一个事件。
考虑到这一点，可以提高检索策略的效率。

###### 2.3）source expansion：提高知识库的覆盖率

wikipedia 和词典资源不能 cover 一个 topic 相关的所有事实，所以引入了其他数据源。

这里提出了一个 algorithm。不展开。

---
#### 3. Automatic knowledge extraction from documents

这篇是关于从大量的 unstructured data 中自动抽取出 structured data 的一个实现（PRISMATIC）。

主要分为两个阶段：第一步，自动抽取 shallow knowlege；第二步，从 shallow knowlege 推断出其他的语义信息。

文章内容主要是 PRISMATIC 实现的细节和它在 WATSON 中的应用。略读。

---
#### 4. Finding needles in the haystack: Search and candidate generation

这篇是关于怎么从大规模数据中检索答案，分为 search 和 candidate generation 两部分。

###### 4.1）search
###### 4.1.1）search unstructured resource
不同的 source 的数据有不同的特点（例如之前提过的 wikipedia article 和 news article）。
作者观察 qa 对和相关的 document 之间的关系，总结出三种类型，对应不同的 search 方式：
* title 是 answer
* title 是 question
* title 既不是 answer 也不是 question

第一种类型，采取 Document search 的方法；后两种类型，采取 Page Search 的方法。

###### 4.1.2）search structured resource
利用的 structured resource 有两个来源：现存的知识库，例如 DBpedia；自建的知识库，PRISMATIC；

###### 4.2）candidate generation
本文讨论的 candidate generation 是针对 unstructured resource 的 search results 的。这可能是因为 structured resource 中检索到的结果不需要进一步处理。

有三种方式（和 4.1.1 中提到的不同的 search 方式对应）：
* Title of Document candidate generation - 对应 Document search 的 output
* Wikipedia Title candidate generation - 对应 Page search 的 output
* Anchor Text candidate generation - 以上两种 search 的 output
