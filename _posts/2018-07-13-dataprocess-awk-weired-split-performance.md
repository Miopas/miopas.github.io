---
layout: post
title: awk 切分列遇到的奇怪的问题
categories: [data]
description: some word here
keywords: data
---

一个特殊符号引发的血案。


前段时间用 awk 处理数据的时候，遇到一个奇怪的事情。

![dataprocess02_1](https://github.com/Miopas/miopas.github.io/raw/master/assets/images/posts/dataprocess02.png)


我希望从 `dev0611.txt` 中统计只有三列的数据和剩下的其他数据。但是按这样写，得到的输出文件合起来的总行数比原文件少了很多。当时遇到这个问题比较匆忙，然后写了个 Python 脚本来做，发现没有问题，就没有再深究了。

然而，事实证明，是坑，总会再踩的。

这几天在处理另一个文件的时候，又遇到了 awk 切分出来的文件行数对不上的事儿。通过 python 脚本的输出和 awk 脚本的输出，发现了一个特殊符号：

![dataprocess02_2](https://github.com/Miopas/miopas.github.io/raw/master/assets/images/posts/dataprocess0202.png)

没错，就是这个神奇的 `^@` 造成的。

~~那一天，人类终于回想起，被 `^M` 支配的恐惧。~~

解决办法在这里：[How to remove this symbol “^@” with vim?](https://superuser.com/questions/75130/how-to-remove-this-symbol-with-vim)

顺便一提，每次遇到这种特殊符号问题，搜索引擎就非常不配合啊！这种时候，可以尝试用符号的英文名去搜索，例如上面这个链接，就是用 `vim invisible symbol hat` 这个 query 搜到的。非常神奇。
