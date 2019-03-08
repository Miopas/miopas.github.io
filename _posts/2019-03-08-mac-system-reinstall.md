---
layout: post
title: Mac 重装系统出现错误“Could not create a preboot volume for APFS install”
categories: [Mac]
description: some word here
keywords: keyword1, keyword2
---

新 Mac 入手发现无法开机，体验了一把重装系统的过程。

首先，开机后按下 `Command + R` 启动系统，会进入这样一个 `macOS Utilities` 界面：![2019-03-08-mac-system-reinstall-pic1](https://github.com/Miopas/miopas.github.io/raw/master/assets/images/posts/2019-03-08-mac-system-reinstall-pic1.png)

一般来说，这时候只要连上 wifi，然后选择第二个选项 `Reinstall macOS` 开始重装就可以了。但是重装过程中报错 `"Could not create a preboot volume for APFS install"`。

查了一圈，最后参考 Quora 上的[这个回答](https://www.quora.com/How-do-I-fix-Could-not-create-a-Preboot-Volume-for-APFS-insatll-on-Mac-OS/answer/Manish-Sharma-2840)解决了。具体步骤如下（需要有网络）：

* Step 1. 按下 `Command + R` 开机后，进入 Disk Utils 把所有的 Volume 都删掉，只留下 Recovery 的 Volume（这是删不掉的，一般来说也看不到）。

* Step 2. 重启 Mac，按下 `Option + Command + R` 启动系统，这时候会看到一个地球图标（不是苹果图标）。开机后，再次进入 Disk Utils，把硬盘再次 `Erase`，`Format` 选择 `OS X Extended (Journaled)`。这里不同系统版本显示的可能会不一样，总之不要选 `APFS` 相关的就对了。

* Step 3. 回到 `macOS Utilities`，选择 `Reinstall macOS` 开始重装。


总体来说 Mac 装系统的体验还是比 Windows 装系统的体验要好多了。另外，这次又一次森森地感受到 Google 是第一生产力。

