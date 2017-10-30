---
layout: post
title: hadoop 踩坑记录（一）
categories: [hadoop, 数据处理]
description: 初次学习使用 hadoop 的一些记录
keywords: hadoop, shell, 数据处理
---

最近开始使用 hadoop，踩到了一些坑，记录下心得。

这个任务是处理一个音乐实体词表，以 `歌曲名 + 歌手名` 为 key 做去重。

------
#### 踩的坑

**概括一下就是 sort 的编码支持问题。**

*下面是场景还原。*

用 mapreduce 脚本跑完以后，我从结果中取了部分样本，用以下脚本检查是否去重成功。
```
#第一列为歌曲名，第二列为歌手名
cat result | cut -f1,2 | sort | uniq -c | sort -k2nr
```
然后发现其中一部分的 count 结果不是 1 。

我首先怀疑了是 hadoop streaming 的参数设置不对，导致在分桶的时候 key 相同的数据没有被分在一个 partition 中。但是用网上的简单的例子检测了一下脚本，参数设置没问题。

然后我检查了 mapreduce 脚本的逻辑，在本地构造了和源数据格式相同的测试样例测试之后，发现逻辑也没问题。

接着我怀疑是 hadoop 集群环境的编码问题，于是我把本地的测试文件 put 到 hdfs 后测试，发现也没问题。

我从结果中抽取了更多的样本，取出其中 count 数目大于 1 的部分来看，然后发现基本上都是包含日文和韩文的条目。
把相应的条目在原文件（result 文件）中用 grep 检查了一下，发现确实只有一个。

之后尝试改进脚本为 
```
cat result | cut -f1,2 | LC_ALL=C sort | uniq -c | sort -k2nr
```
还是没有成功。

**最后用 awk 写了个 count 脚本解决了。**
```
cat result | awk -F"\t" '{key=$1"\t"$2; c[key]++} END {for (i in c) print c[i],i}'
```
（然而由于我 shell 编程水平十分烂，awk 脚本有个bug，又在这个坑里扑腾了半天）

分析了一下原因，可能是 Shell 的 sort 内部对编码的处理有问题，有时间进一步研究一下。

------
#### 总结

**1.用不熟悉的工具处理数据前，先用简单的数据案例测试。（例如这次 hadoop streaming 的排序相关参数）**


**2.注重单元测试。**

按照数据源的数据格式来构造用于单元测试的小文件，尽可能覆盖 mapreduce 中的所有处理逻辑（排序，去重，按日期取最新等）。


**3.在怀疑 hadoop 集群环境之前先怀疑自己的 mapreduce 逻辑**

这次爬坑过程中各种把 hadoop 怀疑了一遍，最终都验证了 hadoop 毫无问题，愚蠢的是我。

不过用小文件本地测，然后把小文件放到 hadoop 集群再测一次依然是个可取的办法。 


------
#### 一个 hput 的小 tip

把从 hadoop 上面 `hadoop fs -getmerge` 下来的结果文件，从本地 `hadoop fs -put` 文件到远端的时候，会莫名的慢。

各方查阅了一下，是因为 `hadoop fs -getmerge` 会在本地产生一个 `.crc` 文件，在进行 `hadoop fs -put` 操作的时候，会先用这个文件进行校验。只要把这个文件删除了就快很多啦。（但是不知道会不会有什么负面影响）


