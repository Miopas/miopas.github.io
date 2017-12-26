---
layout: post
title: Deep parsing in Watson 论文理解
categories: [问答系统]
description: 关于 Watson 中的 Deep parsing 技术
keywords: 问答系统, chatbot, QA system
---

一般的 Parsing，是对自然语言文本做句法层面的解析。Watson 中提出的 Deep parsing，在句法分析的基础上，还做了语义层面的解析，例如 `authorOf` 这样的关系。

这篇 paper 中主要介绍了两个组件：
* English Slot Grammar(ESG)
* Predicate-argument structure(PAS)

`ESG` 的解析结果是一个依存树，称为 `SG tree`。`PAS` 则是在这个树的基础上做了简化和抽象，便于粗粒度的分析。

---
#### 1）English Slot Grammar(ESG)
ESG 的作用，是把自然语言文本的 text 转换成结构化的文本表示的 SG tree。

以下是 pipeline：

Tokenization and Segmentation → 

Morpholexical Analysis → 

Syntactic Analysis

首先对文本进行分词，然后进行词形分析，最后做句法分析。

下面介绍 SG Tree 的一些细节和句法分析的实现。

###### 1.1）SG Tree
SG Tree 是一个依存树（dependency tree）。树的节点由几部分组成：
* A headword `N`：中心词。
* modifier `M`：中心词 `N` 的修饰词。
* slot：预先定义的语法关系，例如 `subj`。每个 `M` 都能填入 `N` 的一个 slot。
* node ID：用于标识 node。
* node features：中心词 `N` 的一些特征，如词性，语义等。见 [Figure 1](https://github.com/Miopas/miopas.github.io/blob/master/_posts/deep_parsing_in_watson_figure_1.jpg) 的最右列。

在 [Figure 1](https://github.com/Miopas/miopas.github.io/blob/master/_posts/deep_parsing_in_watson_figure_1.jpg) 的例子中:
> 以 `chandelier` 为中心词的节点，填入以 `but` 为中心词的节点的 `subj` 的 slot。

###### 1.2）slot 有两类：complement slot 和 adjunct slot
这部分涉及到一些语言学的东西，看不懂，略过。

###### 1.3）surface structure 和 deep structure
SG Tree 中包含了两种信息：`surface structure` 和 `deep structure`。

`surface structure` 指的是语法（grammatical）层面的依存关系；`deep structure` 指的是语义逻辑（logical）层面的依存关系。

*从修饰词推出的 modifier tree structure 是 surface structure。*

观察 [
ure 1](https://github.com/Miopas/miopas.github.io/blob/master/_posts/deep_parsing_in_watson_figure_1.jpg) 的例子：
> 在 deep structure 中，chandelier 是 look，do，use 的 subj；
> 但是在 surface structure 中，chandelier 仅仅是 but 的 subj（[Figure 1](https://github.com/Miopas/miopas.github.io/blob/master/_posts/deep_parsing_in_watson_figure_1.jpg) 中只有 but 和 chandelier 在一条线上）。

###### 1.4）SG 的词典系统
这部分是关于 SG lexical system 的构建。SG Analysis 的过程是**词典驱动**的。
因为 fill slot 的时候需要用到词典，而后续的 syntactic analysis 的主要步骤是 slot-filling。

*这里介绍了词典中 entry 的结构，不展开。*

这里介绍了优化 ESG 的 lexcial system 的几个策略：

* 在匹配的过程中，用动词形式来匹配对应的名词（名词的动词化）：
> 例如：celebration → celebrate

* 引入 WordNet 来扩展 base 词典

* 利用名词-动词关系来扩展 base 词典

*这个关系在 parsing 的过程中不会用到，但是会呈现在 ESG 的输出结果中，供后续的流程使用。*

* 引入 chunk lexicon
chunk lexicon 是包含 multiword 实体的词典。

以下释义摘抄自[网络](https://www.teachingenglish.org.uk/article/lexical-chunk)：
> A lexical chunk is a group of words that are commonly found together. Lexical chunks include collocations but these usually just involve content words, not grammar.
> Example
> In this dialogue there are five possible chunks:
> - Did you stay long at the party?
> - No, I got out of there as soon as they ran out of food.

* 计算出一个分数，后续作为 LAT 的一个 feature
*LAT 见 Question analysis 那篇 paper*

###### 1.5）SG Syntactic Analysis
这部分描述了实现的过程。
**TODO**

#### 2）Predicate-argument structure(PAS)
###### 2.1）介绍
SG tree 是细粒度的，但是后续的分析过程中很多是粗粒度的。
因此，PAS 在 SG tree 的基础上删除了一些对粗粒度的分析不 essential 的部分。好处是更灵活，需要更少的语言学的知识。

举个例子说明。
> I heard that Edison invented the phonograph in 1877.
> I heard that Edison invented a phonograph in 1877.
> I heard Edison invented the phonograph in 1877.
> I heard that Edison was inventing the phonograph in 1877.
> I heard that the phonograph was invented by Edison in 1877.
上述句子的意思有差别，在 SG Tree 的表示中也有差别。但是对于 PAS 来说，上述句子有相同的结构。
例如，“Who invented the photograph？” 这样的 question，如果与其中一句的 PAS 表示相匹配，则也能与其他几句话匹配。这是 PAS 的一个最大的优点；同时也有缺点，可能会产生错误的结论。比如上述句子中的第二句，不能作为 answer 的依据。

###### 2.2）实现
**TODO**

###### 2.3）Pattern-based relation extraction
这部分介绍了基于 pattern 的关系抽取。这里的关系是分领域的、语义层面的关系，例如 `authorOf` 这样的关系。
对于原始的文本，用 pattern 去做 relation extraction 是不现实的，但是对于 ESG 和 PAS 输出的结构化文本，用 pattern 来做关系抽取是可行的。
基于原始的文本写的 pattern，称为 linear pattern；基于 ESG 和 PAS 处理后的文本上写的 pattern，称为 structual pattern。
这里详细说明了为什么 structual pattern 优于 linear pattern，略。

###### 2.4）总结
ESG 和 PAS 是整个系统中比较重要的部分，在后续的一些过程中都会用到 ESG 和 PAS 的结果，例如 Question  Analysis 和 Passage scoring 等。
因此也比较仔细地读了这一篇。
文中的 TODO 的部分，等梳理完全的论文之后找时间补全。


