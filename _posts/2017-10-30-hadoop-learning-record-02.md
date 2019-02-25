---
layout: post
title: Hadoop Streaming 踩坑记录（二）| Python 脚本使用第三方库
categories: [hadoop]
description: 初次学习使用hadoop的一些记录
keywords: hadoop, shell, 数据处理
---


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


参考：

[hadoop streaming python 添加module的问题](http://yuezhilei.com/2017/08/31/hadoop-streaming-python/)

[Hadoop Streaming和Spark的一些坑](https://jayveehe.github.io/2017/05/06/hadoop-spark-notes/)

[在Hadoop Streaming中使用PyPy](https://gist.github.com/codeb2cc/13ec2399cc1c95773eaa)

[使用Hadoop的MapReduce和jieba分词统计西游记中的词频](https://www.lylinux.net/article/2017/11/4/31.html)

