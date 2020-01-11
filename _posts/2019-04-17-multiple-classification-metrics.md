---
layout: post
title: 计算多分类任务中 precsion、recall、F1 指标
categories: [MachineLearning]
description: some word here
keywords: keyword1, keyword2
---

关于如何计算混淆矩阵和借助混淆矩阵来计算 precision、recall、F1。并比照二分类任务，描述在多分类任务的场景下如何计算 precsion、recall 和 F1。

## 混淆矩阵和二分类任务指标
在评价一个分类器的性能的时候，通常用到的指标是 precsion、recall 和 F1 值，并通过混淆矩阵来辅助计算这些值。<b>混淆矩阵(Confusion matrix)</b>又称为错误矩阵，在机器学习中被用于计算分类器的评价指标。对于一个分类器的预测结果，可以画出这样一个混淆矩阵：表格每一行表示预测的类别，每一列表示真实的类别。在二分类任务中，混淆矩阵如下所示：

|| label = 1 | label = 0 |
| :------: | :------: | :------: |
| predict = 1 | TP | FP |
| predict = 0 | FN | TN |


其中 label 是数据的标注标签，predict 是分类器的预测结果。为了计算 precsion、recall 和 F1，这里引入了 4 个概念：
* <b>TP</b>: True Positive，表示 label = 1 且 predict = 1
* <b>FP</b>: False Positive，表示 label = 0 且 predict = 1
* <b>FN</b>: False Negative，表示 label = 1 且 predict = 0 
* <b>TN</b>: True Negative，表示 label = 0 且 predict = 0

在记忆这 4 个概念的时候可以这样理解：True/False 表示预测结果和标注标签是否一致，Positive/Negative 表示预测结果是 1 还是 0。

得到混淆矩阵后，计算 precsion、recall 和 F1 值只需要代入公式：

<p align="center">precsion =  TP/(TP + FP)</p>
<p align="center">recall   =  TP/(TP + FN)</p>
<p align="center">F1   =  2 * precsion * recall/(precsion + recall)</p>

举例说明整个计算过程。有 5 条数据的标签和预测结果如下所示：

<p align="center">labels   = ['class_A', 'class_B', 'class_A', 'class_A', 'class_B']</p>
<p align="center">predicts = ['class_A', 'class_A', 'class_B', 'class_B', 'class_B']</p>

可以得到混淆矩阵：

|| label = class_A | label = class_B |
| :------: | :------: | :------: |
| predict = class_A | TP = 1 | FP = 1 |
| predict = class_B | FN = 2 | TN = 1 |


计算 precsion、recall、F1：

<p align="center">precsion =  1/(1 + 1) = 0.50</p>
<p align="center">recall   =  1/(1 + 2) = 0.33</p>
<p align="center">F1   =  2 * 0.50  * 0.33/(0.50 + 0.33) = 0.40</p>



## 多分类任务的 precision、recall、F1
在多分类任务中计算 precsion、recall、F1 的时候，通常是将 n 个类看做 n 个二分类任务来计算。以三分类任务为例，我们可以画出这样一个混淆矩阵：

|| label = class_A | label = class_B | label = class_C |
| :------: | :------: | :------: | :------: |
| predict = class_A | | | |
| predict = class_B | | | |
| predict = class_C | | | |


这时候需要对每个类单独处理填表。对于 `class_A` 类：

||label = A|label = B or label = C|
| :------: | :------: | :------: |
|predict = A|TP(A)|FP(A)|
|predict = B or predict = C|FN(A)|TN(A)|

对于 `class_B` 类：

||label = B|label = A or label = C|
| :------: | :------: | :------: |
|predict = B|TP(B)|FP(B)|
|predict = A or predict = C|FN(B)|TN(B)|

对于 `class_C` 类：

||label = C|label = A or label = B|
| :------: | :------: | :------: |
|predict = C|TP(C)|FP(C)|
|predict = A or predict = B|FN(C)|TN(C)|

这样就可以计算各类的 TP、FP、TN、FN 的值。

得到混淆矩阵后，在计算多分类整体的 precision、recall、F1 的时候有多种方法。常见的有 `Macro` 和 `Micro` 方式。这里搬运知乎的[对多分类数据的模型比较选择，应该参考什么指标？ - 谢为之的回答 - 知乎](https://www.zhihu.com/question/51470349/answer/439218035)的解释：
> Macro F1: 将n分类的评价拆成n个二分类的评价，计算每个二分类的F1 score，n个F1 score的平均值即为Macro F1。
> Micro F1: 将n分类的评价拆成n个二分类的评价，将n个二分类评价的TP、FP、RN对应相加，计算评价准确率和召回率，由这2个准确率和召回率计算的F1 score即为Micro F1。
> 一般来讲，Macro F1、Micro F1 高的分类效果好。Macro F1受样本数量少的类别影响大。

（“将n个二分类评价的TP、FP、RN对应相加”疑似typo，应该是 TP、FP、FN。）

在数据各类样本不均衡的情况下，采用 Micro F1 较为合理。

## 参考
[Confusion Matrix](https://en.wikipedia.org/wiki/Confusion_matrix)

