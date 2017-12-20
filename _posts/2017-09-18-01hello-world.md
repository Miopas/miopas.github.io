---
layout: post
title: Hello World
categories: [杂谈]
description: 惯例的Hello World
keywords: 杂谈
---


首先十分感谢 [@mzlogin](https://github.com/mzlogin/mzlogin.github.io) 提供了这个模板，好看且实用。

开源真好，感谢开源。

开这个博客主要是为了记录一些实践中遇到的技术问题，希望自己能认真写，坚持写。


___


#### 顺便记录一个搭建blog过程中遇到的一个小问题

一开始我是按照 [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html) 的流程，新建了一个名为 `blog` 的 repo ，用 `gh-pages` 分支的方式来搭建。

然后从 `mzlogin.github.io` 迁移配置文件，修改成我自己的配置的时候出了问题：**首页可以正常显示，但是页面中的 url 都不能正常 link。**

*例如: `首页` 按钮的正常链接的 url 应该是 `https://miopas.github.io/blog/`，点击 `首页` 的时候，跳转的页面的链接是 `https://miopas.github.io/`。*

尝试各种修改了 `_config.yml` 中的配置都无效果。最后还是选择了使用 `master` 分支的方式来搭建，一切正常。

之后如果有机会的话想研究下相关的文档。

