---
layout: post
title: PaddlePaddle(v2) 安装及运行的踩坑记录
categories: [PaddlePaddle]
description: 虽然感觉这篇笔记大概永远都用不上了但是写吧
keywords: PaddlePaddle
---

关于 PaddlePaddle(v2) 安装过程的一些折腾。注意现在 PaddlePaddle 已经全面升级到 fluid 版本，这篇笔记不一定适用于最新版本。

## 背景
PaddlePaddle 是百度开源的 Deep Learning 框架，其定位应该是类似于 TensorFlow，优点是有相对完善的中文文档以及在 Github 上面可以中文交流。不过国内用的人好像不是很多。

因为我有段时间要实现一个 DSSM 模型，想起来有人推荐过 PaddlePaddle 有一个 DSSM 的官方实现模型，所以折腾了一下 PaddlePaddle。

## 安装和运行

*以下命令在操作系统 CentOS 7.x 环境下运行，Python 版本均为 2.7*

#### 通过 pip 安装
如果运行环境是 CPU 环境，直接用 `pip` 安装即可。不过在我的安装环境中发生了一个 error：
```js
Install err:
_posixsubprocess.c:16:20: fatal error: Python.h: No such file or directory
#include "Python.h"
^
compilation terminated.
error: command 'gcc' failed with exit status 1
```
通过安装 `python-devel` 解决，运行：
```console
yum install python-devel
```

#### 安装 GPU 环境运行版本

一开始我依然使用 `pip` 来安装，安装成功后在运行模型的时候报错：
```
ImportError: libmklml_intel.so: cannot open shared object file: No such file or directory
```
安装 `mkl` 解决。参考: [wheels](https://github.com/mind/wheels#mkl)

然后再次运行，再次报错：
```
Check failed: cudaSuccess == cudaStat (0 vs. 35) Cuda Error: CUDA driver version is insufficient for CUDA runtime version
```
这里的错误信息表明，CUDA 运行版本和实际安装的 CUDA 驱动版本不一致。由于由于运行版本和 PaddlePaddle 的实现有关，显然我不能改运行版本。因此最直接的办法是变更驱动版本使其一致。但是我当时可能是脑子抽了，就是不想动驱动版本，加上 PaddlePaddle 官方提供了 docker 方式安装，我想或许有办法可以不用动服务器的驱动，于是从这里开始了我长达 1.5 天的 `Google → 解决 error → Google → 解决 error` 的循环。

结果是所有基于 docker 的尝试都失败了。事实上，虽然 docker 镜像内提供了和 PaddlePaddle 版本一直的 CUDA，但是这仍然是 runtime version。CUDA driver version 是由 host machine 提供的，在运行 nvidia-docker 的时候会 mount 到 docker 环境中。而 CUDA runtime verion 是由 docker 镜像提供的，只要 host machine 的驱动版本没变化，永远都是不兼容。

最后还是通过把 CUDA 驱动版本降级解决了。具体过程见：[CentOS 更新 CUDA 驱动版本从 9.1 降级为 9.0](https://miopas.github.io/2018/08/13/nvidia-driver-downgrade/)。

下面是（虽然什么卵用的）解决掉的 error 的记录。

| 错误信息   | 解决方法    | 说明   | 参考 |
| :--------   | :-----   | :---- | :---- |
| Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.29/images/create?fromImage=docker.paddlepaddlehub.com%2Fpaddle&tag=latest: dial unix /var/run/docker.sock: connect: permission denied        | sudo gpasswd -a guoyuting docker<br> sudo service docker restart       |   发生在尝试 docker pull 的过程中    | https://blog.csdn.net/qiyueqinglian/article/details/50952870 |
| Can’t find libcublas.so        | add the path  /usr/local/cuda/lib64 to LD_LIBRARY_PATH |   -    | https://github.com/tensorflow/tensorflow/issues/15604 |
| F0608 17:17:03.145012  1617 DynamicLoader.cpp:104] Check failed: nullptr != *dso_handle Failed to find dynamic library: libcudnn.so (libcudnn.so: cannot open shared object file: No such file or directory)        | 见参考 | 划重点：CUDA library is different from CUDNN Library |    https://github.com/tensorflow/tensorflow/issues/4827 |
|create nvidia_driver_390.46: Error looking up volume plugin nvidia-docker: legacy plugin: plugin not found. | sudo nvidia-docker-plugin | 尝试使用 NVIDIA-docker run | https://github.com/NVIDIA/nvidia-docker/issues/437 | 
|Error: Could not load NVML library|export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/var/lib/nvidia-docker/volumes/nvidia_driver/384/lib/| - |https://github.com/NVIDIA/nvidia-docker/issues/74|
|CUDA error: invalid device function| - | 也是因为 CUDA driver version 和 runtime version 不匹配。| - |


## 后记
回头想想这次是被自己坑了一把，早点把驱动版本改下就完事儿了。但另一方面，我发现对于一个开源软件来说“社区”确实是非常重要。

在搜索 GPU 环境的问题的过程中，很多 solution 我是在 TensorFlow 的 issue 中找到的。TensorFlow 无疑在社区建设上是领先于其他几个深度学习框架的。对于开源软件来说，良好的社区能带来充足的讨论，从而带动更多的开发者提供自己的模型实现，然后开源软件本身又通过开发者提出的问题进一步完善。而一个冷门的开源软件常常都没人提问，代码中有 Bug 也没人发现，开发者遇到问题很多时候只能自己啃源码，确实比较痛苦了（之前使用 Deeplearning4j 也有同样的感触）。之后我在选择开源软件的时候也会先考虑社区是否完善，少给自己挖坑。

## 参考
[PaddlePaddle 0.12.0 (v2) 官方文档（可能某天就失效了）](http://staging.paddlepaddle.org/documentation/docs/zh/0.12.0/getstarted/index_cn.html)

[CUDA driver version is insufficient for CUDA runtime version 的解决方案](https://zhuanlan.zhihu.com/p/28954367)

[NCCL compilation and linking version not match ](https://github.com/PaddlePaddle/Paddle/issues/8195)

[更新 paddlepaddle 0.11.0 出错 CUDA error: invalid device function ](https://github.com/PaddlePaddle/Paddle/issues/10062)

[Paddle 指定 GPU](https://github.com/PaddlePaddle/Paddle/issues/6725)

[Download cuDNN via wget or curl?](https://devtalk.nvidia.com/default/topic/1002046/cuda-setup-and-installation/download-cudnn-via-wget-or-curl-/)
