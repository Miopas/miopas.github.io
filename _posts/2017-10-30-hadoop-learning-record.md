---
layout: post
title: hadoop 踩坑记录（二）
categories: [hadoop]
description: 初次学习使用hadoop的一些记录
keywords: hadoop, shell, 数据处理
---

记录最近新踩的两个坑。

1. hadoop streaming 的 python 脚本使用第三方库
2. 以数值列为唯一的 key 来排序

------
#### 1.hadoop streaming 的 python 脚本使用第三方库

由于公司的 hadoop 集群部署的机器上的 python 版本太低，缺少很多基础库，需要把本地的 python 打包一个传上去，用 `setCacheArchive` 参数来引用。
参数示例如下：
```
-mapper  "python/python27/bin/python mapper.py"  -file mapper.py \
-cacheArchive /xx/xx/python27.tar.gz#python \
```

之后如果需要更新 python 库，可以把 hadoop 上打包好的 `python27.tar.gz` 下载到本地，解压。
然后把本地的 python 安装目录下的 `lib/python2.7/site-packages` 需要的库拷贝到解压的这个目录下对应的 `lib` 路径。
最后把这个目录重新打包，上传到 hadoop 就搞定了。

**2017-11-08 更新：有些情况下，直接拷贝 `site-packages` 会有问题。更合适的做法是用 `pip` 重新安装对应的第三方库，安装到指定目录下。**


参考：[hadoop streaming python 添加module的问题](http://yuezhilei.com/2017/08/31/hadoop-streaming-python/)

------
#### 2.以数值列为唯一的 key 来排序的问题

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
-jobconf stream.num.map.output.key.fields=2 \
-jobconf mapred.text.key.comparator.options='-k2nr' \
-jobconf num.key.fields.for.partition=1
```

分析原因，是必须让需要排序的数据分在**同一个** `partition` 里才能正确排序。

`错误尝试 A`和`错误尝试 B`中，分别以`歌名`和`热度`作为 partition 划分的 `key`，不可能让所有的数据都分在一个 `partition` 中。
