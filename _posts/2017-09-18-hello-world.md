---
layout: post
title: Hello World——博客搭建小记
categories: [杂谈]
description: 惯例的Hello World
keywords: 杂谈
---

开源真好，感谢开源。

首先十分感谢 [@mzlogin](https://github.com/mzlogin/mzlogin.github.io) 提供了这个模板，好看且实用。开这个博客主要是为了记录一些实践中遇到的技术问题，希望自己能认真写，坚持写。

整体教程参考博文[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)，在搭建过程中遇到的一些问题在此略加记录。
___


## URL 路径问题
一开始我按照教程的流程走，新建了一个名为 `blog` 的仓库，用 `gh-pages` 分支的方式来搭建，然后从 `mzlogin.github.io` 迁移配置文件。当需要修改成我自己的配置的时候出了问题：首页可以正常显示，但是页面中的 url 都不能正常 link。例如: `首页` 按钮的正常链接的 url 应该是 `https://miopas.github.io/blog/`，点击 `首页` 的时候，跳转的页面的链接是 `https://miopas.github.io/`。 尝试各种修改了 `_config.yml` 中的配置都无效果。最后改为使用 `master` 分支的方式来搭建，一切正常。


## 评论模块的 URL 路径
在设置评论模块的配置后，测试阶段出现“无法找到链接”的错误。原因是在 gitalk 模块发送请求的时候使用的 url 是 `miopas.github.io`，而实际上需要用 `OAuth Apps` 中的 `Authorization callback URL`。而在当初搭建博客的时候，这个地址被设置为 `https://miopas.github.io/blog/`。修改为 `https://miopas.github.io/` 即可。
