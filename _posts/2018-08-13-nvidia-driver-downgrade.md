---
layout: post
title: CentOS 更新 CUDA 驱动版本从 9.1 降级为 9.0
categories: [Linux]
description: some word here
keywords: keyword1, keyword2
---


CUDA 的版本有两种，一种是 `runtime version`，一种是 `driver version`，其中 `driver version` 是由所安装的 nvidia 驱动版本所决定的。

如果仅需要修改 `runtime version`，不需要卸载 nvidia 驱动。可以参考教程：[CentOS 7 卸载CUDA 9.1 安装CUDA8.0 并安装Tensorflow GPU版](http://whatbeg.com/2018/03/17/cudainstall.html)。这里我需要把 CUDA 的**驱动**版本也降级，所以需要重新安装对应的 nvidia 驱动。

在开始之前，先看看这个教程：[Install NVIDIA Driver and CUDA on Ubuntu / CentOS / Fedora Linux OS](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07)，确认依赖环境已经准备好。

首先从[官网](https://www.nvidia.cn/Download/index.aspx?lang=cn)找到需要的 nvidia driver 驱动版本。

我需要的驱动版本是 `384.145`，但是在官网找不到 `runfile` 版本，只有 `rpm` 版本。（由于时间有点久了，我不记得为什么只能用 `runfile` 来安装，印象中 `rpm` 安装有一些问题。 ）好在找到了一个民间大佬提供的[脚本](https://raw.githubusercontent.com/brunsgaard/kops-nvidia-docker-installer/master/nvidia-docker-installer.sh)，其中可以找到需要的 URL 地址。

另外，如果希望通过 `wget` 来下载 nvidia 官网的安装包，可以用 `Link Redirect Trace Extension` 这个 Chrome 插件来找到对应的 URL 地址。

然后，运行 `runfile` 文件安装 nvidia 驱动，出现了一个错误：
```
ERROR: An NVIDIA kernel module 'nvidia' appears to already be loaded in your kernel.  This may be because it is in use (for
       example, by an X server, a CUDA program, or the NVIDIA Persistence Daemon), but this may also happen if your kernel was
       configured without support for module unloading.  Please be sure to exit any programs that may be using the GPU(s) before
       attempting to upgrade your driver.  If no GPU-based programs are running, you know that your kernel supports module
       unloading, and you still receive this message, then an error may have occured that has corrupted an NVIDIA kernel module's
       usage count, for which the simplest remedy is to reboot your computer.

```
这是因为系统中已经有 nvidia 驱动在运行着了。所以在安装新的驱动之前，要先解决就旧的驱动。参考 [NVIDIA module already loaded in kernel](https://codeyarns.com/2017/09/04/nvidia-module-already-loaded-in-kernel/)，运行：
```shell
echo 'blacklist nvidia' > /etc/modprobe.d/blacklist.conf
bash NVIDIA-Linux-x86_64-xxx.xx.run --uninstall  # uninstall 这一步根据具体情况不同
reboot
```
重启后再安装一次，显示 `driver is installed successfully`。运行 `nvidia-smi` 检查 nvidia 驱动版本：
![pic01](https://github.com/Miopas/miopas.github.io/raw/master/_posts/nvidia-driver-01.png)

运行 `/usr/local/cuda/samples/1_Utilities/deviceQuery/deviceQuery` 检查 CUDA 版本：
![pic02](https://github.com/Miopas/miopas.github.io/raw/master/_posts/nvidia-driver-02.png)

## Reference
[CentOS 7 卸载CUDA 9.1 安装CUDA8.0 并安装Tensorflow GPU版](http://whatbeg.com/2018/03/17/cudainstall.html)

[CentOS 7 安装 NVIDIA 显卡驱动和 CUDA Toolkit](https://blog.csdn.net/xueshengke/article/details/78134991)

[Install NVIDIA Driver and CUDA on Ubuntu / CentOS / Fedora Linux OS](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07)


