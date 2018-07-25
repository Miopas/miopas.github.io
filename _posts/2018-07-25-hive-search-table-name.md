---
layout: post
title: Hive 根据 HDFS 路径查找表名
categories: [Hadoop, Hive]
description: some word here
keywords: keyword1, keyword2
---

我们都知道，当表名已知的时候，可以用 `hive -e "desc formatted [表名]"` 来获取表的信息，从而得知表的 HDFS 路径。那么反过来，已知 HDFS 路径的时候，怎么查到这个目录下建了什么样的表呢？

各种 Google 了一下，Hive 似乎没有原生的语句支持这一的查询。所以我想了一个比较笨的办法：*把所有的表的定义打印出来，从中搜索 HDFS 路径。*

首先，把 `default` 数据库中的所有表名保存到文件 `table_name.txt`：
```shell
hive -e "show tables from default" 1>table_name.txt
```

然后，遍历 `table_name.txt`，打印每个表的定义，保存在 `table_definition.txt`：
```shell

# this is a shell script 

while read line
do
    echo "$line"
    eval "hive -e 'desc formatted default.$line'"
done < table_name.txt > table_definition.txt
```

最后，在 `table_definition.txt` 里找到 HDFS 路径。

*Reference:*
[How to get all table definitions in a database in Hive?](https://stackoverflow.com/questions/35004455/how-to-get-all-table-definitions-in-a-database-in-hive)

~~谨以此文献给建完表就忘的小伙伴~~


