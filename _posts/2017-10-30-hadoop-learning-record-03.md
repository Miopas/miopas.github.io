---
layout: post
title: Hadoop Streaming 踩坑记录（三）| 以数值列为唯一的 key 来排序
categories: [hadoop]
description: 初次学习使用hadoop的一些记录
keywords: hadoop, shell, 数据处理
---

以数值列为唯一的 key 来排序的问题。

###### case 描述
```
这是一个根据歌曲热度来排序的 case：给出歌曲和歌曲的热度，按照歌曲的热度排序。

数据示例（格式：歌曲\t热度）：
A	2
B	1
C	3

期望结果（格式：歌曲\t热度）：
C	3
A	2
B	1
```
***
###### 错误尝试 A 

mapper 输出的数据格式为：
```
歌名\t热度
```

hadoop streaming 参数设置为：
```
-jobconf stream.num.map.output.key.fields=2 \
-jobconf mapred.text.key.comparator.options='-k1， -k2nr' \
-jobconf num.key.fields.for.partition=1
```
***
###### 错误尝试 B

mapper 输出的数据格式为：
```
热度\t歌名\t热度
```

hadoop streaming 参数设置为：
```
-jobconf stream.num.map.output.key.fields=1 \
-jobconf mapred.text.key.comparator.options='-k1nr' \
-jobconf num.key.fields.for.partition=1
```
***
###### 正解

mapper输出的数据格式为：`0\t热度\t歌名\t热度`

hadoop streaming 参数设置为：
```
-jobconf mapred.output.key.comparator.class=org.apache.hadoop.mapred.lib.KeyFieldBasedComparator \
-jobconf stream.num.map.output.key.fields=2 \
-jobconf mapred.text.key.comparator.options='-k2nr' \
-jobconf num.key.fields.for.partition=1
```

分析原因，是必须让需要排序的数据分在**同一个** `partition` 里才能正确排序。

`错误尝试 A`和`错误尝试 B`中，分别以`歌名`和`热度`作为 partition 划分的 `key`，不可能让所有的数据都分在一个 `partition` 中。
