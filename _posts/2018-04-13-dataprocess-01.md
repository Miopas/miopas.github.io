---
layout: post
title: 数据处理问题记录
categories: [data]
description: some word here
keywords: data
---

数据处理过程中遇到的奇奇怪怪的问题的处理记录。


1. Vim 去掉 ^M 字符

单个文件：
```
%s/^M//g
```
(^M是ctrl+v,ctrl+m) 
