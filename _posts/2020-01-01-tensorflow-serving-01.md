---
layout: post
title: TensorFlow Serving （一）——模型训练、部署和更新
categories: [TensorFlow]
description: TensorFlow Serving
keywords: tensorflow
---

TensorFlow Serving 是 TensorFlow Extended (TFX) 框架的一部分，其出发点是简化机器学习模型的训练、部署和更新的过程，同时提供更灵活高效的模型管理。本文介绍如何基于 TensorFlow Serving 原生接口进行模型的训练、部署和更新。

## 1. 安装 TensorFlow Serving
TensorFlow Serveing （以下简称 Serving）官方提供了几种不同的安装方式： docker 安装、APT 安装、源码编译。另外存在一种 PIP 安装的方式，其命令为 `pip install tensorflow-serving-api`。但这种方式仅用于安装 Serving 本身，仍需要配合 TensorFlow 本体使用。 如果你写了个 Python 的 client，或许需要依赖这个包的某些 API（比如 gRPC）。出于灵活性考虑，这里选择了 docker 安装的方式。

我的测试的系统环境是 `macOS Mojave 10.14.4`。 根据[官网教程](https://www.tensorflow.org/tfx/serving/docker)，从 docker 仓库中拉取镜像：
```sh
# Download the TensorFlow Serving Docker image and repo
docker pull tensorflow/serving:nightly-devel
```
与教程中不同的是教程安装的是 `:latest` 标签的镜像，而我选择了 `:nightly-devel` 标签的 docker 镜像。`:latest` 的镜像仅满足使用 Serving 的最少依赖，可用于部署已保存的 Serving 的模型（在镜像内官方提供了一些训练好的模型），可以迅速体验一把部署模型的快感（？）。但是我们之后希望能够训练自己的模型，因此选择了供开发者使用的 `:nightly-devel` 标签的镜像，其中包含了一些使用 TensorFlow 训练模型的必要依赖。此外由于当前环境中没有 GPU，这个版本也是无 GPU 支持的版本。

由于我后续使用的模型依赖了其他一些 Python 包，例如 `sklearn` 和 `pandas`，因此我在官方镜像的版本中安装了其他一些包，并保存新的镜像为 `tensorflow/serving:nightly-devel-mypy2`，其完整依赖可见 [requirements.txt](TODO)。因此后续代码中所用镜像均为 `tensorflow/serving:nightly-devel-mypy2`。

## 2. 训练模型、启动服务和测试模型
在官方文档 [Building Standard TensorFlow ModelServer](https://www.tensorflow.org/tfx/serving/serving_advanced) 中介绍了运行 `mnist` 测例的方式。本文也运行了官方示例，详细过程见文末的`附录 A：minist serving 官网示例测试`。这里以 SVM 分类模型为例，介绍如何使用 Serving 进行训练模型、启动服务和测试模型。本节涉及代码均位于 [Miopas/serving](https://github.com/Miopas/serving) 仓库的 `scripts/models` 目录下。

#### 2.1 训练模型
在 TFX 的官方文档中提到这个框架是可扩展的，可以支持非 tf 实现的模型。但进一步阅读文档发现，用 C++ 实现一个 Servable （TFX 中的一个独立提供服务的组件），给这个模型封装一个 tensorflow::Session 的外壳，然后接入 TFX 的框架。这需要对 TFX 的代码有更深入的理解，目前我也没有对源码或文档研究到这个程度，因此选择了基于 TensorFlow 的 SVM 实现。

模型代码来源于 Github 上的 [eakbas/tf-svm](https://github.com/eakbas/tf-svm)。在实验过程中，我在这份代码上进行了一些修改，主要包括以下几点：
* 修改部分涉及 TensorFlow API 的代码使之兼容 TensorFlow 2.0 版本；
* 修改损失函数为 `suquared hinge loss`；
* 数据预处理使用 `data_transformer.py`；
* 模型储存的部分修改为 Serving 兼容的方式。

关于损失函数，源代码用的是 cross-entropy loss，在改动之前模型无法收敛。参考了 sklearn 的 SVM 代码，改为 suquared hinge loss 之后就可以收敛了，不知道为什么 _(:з」∠)_。

*注：由于数据的隐私问题，我这里取用 5 个样本用于示例。原数据的规模大约为 60k 个样本，124 个分类。*

训练 SVM 的命令保存在 `train_svm.sh` 中，其内容如下：
```sh
#!/bin/bash
set -e  -x
docker run -t -v $(pwd):$(pwd) -v /tmp:/tmp tensorflow/serving:nightly-devel-mypy2 \
	bash -c "cd $(pwd)/tf-svm; python train.py \
                --train_data_file=./data/sample.csv  \
                --dev_data_file=./data/sample.csv \
                --export_path_base=/tmp/serving/svm_cls \
                --num_class=5  \
                --batch_size=10 \
                --num_epochs=1 \
                --evaluate_every=1 \
                --checkpoint_every=1 \
                --num_checkpoints=1 \
                --regulation_rate=1e-4 \
                --model_version=1" &
```

部分输出日志如下（为了方便阅读，省略了一些 WARNING 输出）：
```sh
2020-01-11T02:48:14.976371: step 0, loss 0.985851, acc 0

Evaluation:
2020-01-11T02:48:15.026303: step 1, loss 0.817866, acc 0.8

Saved model checkpoint to /Users/gyt/github/serving/scripts/models/tf-svm/runs/1578710894/checkpoints/model-1
...
Saved model to /tmp/serving/svm_cls/1
```

#### 2.2 启动服务和测试
相对于训练过程，启动服务和测试的过程则简单很多，基本就是跟着官方教程走。这里简要说明下运行脚本的过程及其输出。

运行 `serve_svm.sh` 启动服务，代码如下：
```sh
#!/bin/bash

set -e  -x
# 设置模型路径
TESTDATA="/tmp/serving/"

MODEL_BASE_PATH="/models"

# 设置模型名称
MODEL_NAME=svm_cls

PORT=8501

# 启动服务
docker run -t --rm -p 8501:8501 -p 8500:8500 -v $(pwd):$(pwd) -v ${TESTDATA}:${MODEL_BASE_PATH} tensorflow/serving:nightly-devel-mypy2 \
	bash -c "tensorflow_model_server \
		--port=8500 --rest_api_port=8501 \
		--model_name=${MODEL_NAME} \
		--model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME}" &
```

启动成功的日志如下所示：
```sh
models/svm_cls/1/assets.extra/tf_serving_warmup_requests
2020-01-11 02:54:27.584164: I tensorflow_serving/core/loader_harness.cc:87] Successfully loaded servable version {name: svm_cls version: 1}
2020-01-11 02:54:27.609959: I tensorflow_serving/model_servers/server.cc:358] Running gRPC ModelServer at 0.0.0.0:8500 ...
[warn] getaddrinfo: address family for nodename not supported
2020-01-11 02:54:27.616895: I tensorflow_serving/model_servers/server.cc:378] Exporting HTTP/REST API at:localhost:8501 ...
[evhttp_server.cc : 238] NET_LOG: Entering the event loop ...
```

测试服务有 REST 请求和 gRPC 两种方式可选，这里我们使用 REST 请求的方式，并为了方便写了一个 Python 的请求脚本 `client.py`。运行测试脚本 `test_svm.sh`，代码如下：
```
#!/bin/bash

set -e  -x

MODEL_NAME=svm_cls
cd $(pwd)/tf-svm;python client.py http://localhost:8501/v1/models/${MODEL_NAME}:predict
```

测试成功的输出结果如下：
```sh
DEBUG:jieba:Loading model cost 0.463 seconds.
Prefix dict has been built successfully.
DEBUG:jieba:Prefix dict has been built successfully.
DEBUG:DataTransformer:run extract_features
DEBUG:DataTransformer:data size is 1
DEBUG:DataTransformer:data matrix shape is (1, 62)
test:@kg.MutualFund 基金@初始规模 是怎样
{
    "predictions": [
        {
            "predictClass": 0,
            "scores": 0
        }
    ]
}
y_true:[0]
```

先写到这儿，下一次写模型的在线更新和云端部署。

## 附录 A：minist serving 官网示例测试

首先从 Github 上下载 Tensorflow Serving 代码：
```sh
git clone https://github.com/tensorflow/serving.git
```

然后，训练 mnist 分类模型并保存，运行：
```sh
tools/run_in_docker.sh -d tensorflow/serving:nightly-devel-mypy2 python tensorflow_serving/example/mnist_saved_model.py /tmp/mnist
```

终端打印输出如下所示：
```sh
I0102 03:37:29.539124 140639269201728 builder_impl.py:637] No assets to save.
INFO:tensorflow:No assets to write.
I0102 03:37:29.540767 140639269201728 builder_impl.py:457] No assets to write.
INFO:tensorflow:SavedModel written to: /tmp/mnist/1/saved_model.pb
I0102 03:37:29.669203 140639269201728 builder_impl.py:422] SavedModel written to: /tmp/mnist/1/saved_model.pb
Done exporting!
```

查看输出的模型：
```sh
$ ls /tmp/mnist
1
$ ls /tmp/mnist/1
saved_model.pb  variables
```

加载模型，启动服务：
```sh
#!/bin/bash

# 设置模型路径
MODEL_BASE_PATH="/models"

# 设置模型名称
MODEL_NAME=mnist

# 启动服务
docker run -t --rm -p 8500:8500 -p 8501:8501 -v $(pwd):$(pwd) -v /tmp/mnist/:/models/mnist tensorflow/serving:nightly-devel-mypy2 \
        bash -c "tensorflow_model_server \
                --port=8500 --rest_api_port=8501 \
                --model_name=${MODEL_NAME} \
                --model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME}" &
```

服务启动成功，终端打印信息如下：
```sh
2020-01-02 03:41:46.564987: I tensorflow_serving/core/loader_harness.cc:87] Successfully loaded servable version {name: mnist version: 1}
2020-01-02 03:41:46.592615: I tensorflow_serving/model_servers/server.cc:358] Running gRPC ModelServer at 0.0.0.0:8500 ...
[warn] getaddrinfo: address family for nodename not supported
[evhttp_server.cc : 238] NET_LOG: Entering the event loop ...
2020-01-02 03:41:46.622449: I tensorflow_serving/model_servers/server.cc:378] Exporting HTTP/REST API at:localhost:8501 ...
```

测试 Serving 服务：
```sh
$ tools/run_in_docker.sh -d tensorflow/serving:nightly-devel-mypy2 python tensorflow_serving/example/mnist_client.py --num_tests=1000 --server=127.0.0.1:8500
```

结果如下：
```sh
$ tools/run_in_docker.sh -d tensorflow/serving:nightly-devel-mypy2 python tensorflow_serving/example/mnist_client.py --num_tests=1000 --server=0.0.0.0:8500
== Pulling docker image: tensorflow/serving:nightly-devel-mypy2
Error response from daemon: manifest for tensorflow/serving:nightly-devel-mypy2 not found
WARNING: Failed to docker pull image tensorflow/serving:nightly-devel-mypy2
== Running cmd: sh -c 'cd /Users/gyt/github/serving; python tensorflow_serving/example/mnist_client.py --num_tests=1000 --server=0.0.0.0:8500'
Extracting /tmp/train-images-idx3-ubyte.gz
Extracting /tmp/train-labels-idx1-ubyte.gz
Extracting /tmp/t10k-images-idx3-ubyte.gz
Extracting /tmp/t10k-labels-idx1-ubyte.gz
........................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
Inference error rate: 10.4%
```

注意这里使用的官方提供的脚本 `mnist_client.py` 是一个 gPRC 调用，因此使用gRPC 的端口 `8500`，而不是 REST 的端口 `8501`。同时这个脚本依赖于 Serving 的 API，因此要安装对应的包 `pip install tensorflow-serving-api`。



## 附录 B：如何从已有的 docker 镜像重新编译 bin

首先，下载源码：
```sh
git clone https://github.com/tensorflow/serving.git
cd serving
```

然后，运行脚本：
```sh
tools/run_in_docker.sh bazel build -c opt tensorflow_serving/...
```
注：如果这一步报错可参考[附录 C：bazel 编译报错](TODO)。这一步会重新编译出二进制文件，编译完成的文件在当前目录的 `bazel-bin/tensorflow_serving/model_servers/` 目录下，其中就有后续需要用的 `tensorflow_model_server` 这个 bin。

最后，用新的 bin 启动服务:
```sh
# 设置模型路径
TESTDATA="$(pwd)/tensorflow_serving/servables/tensorflow/testdata"

# 设置模型名称
MODEL_NAME=half_plus_two

# 启动服务
# 挂载了当前目录，使用 bazel-bin 目录下的新的 bin 来启动
docker run -t --rm -p 8501:8501 -v $(pwd):$(pwd) tensorflow/serving:nightly-devel bash -c "cd $(pwd);bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server --port=8500 --rest_api_port=8501   --model_name=${MODEL_NAME} --model_base_path=${TESTDATA}/${MODEL_NAME}" &
```

## 附录 C：bazel 编译报错
执行这一步发生如下报错：
```sh
tools/run_in_docker.sh bazel build -c opt tensorflow_serving/...

ERROR: /home/guoyt/serving/.cache/_bazel_guoyt/0456a77278cd25a6cabf8d2d5a95c9b3/external/com_google_protobuf/BUILD:148:1: no such package '@zlib//': The repository '@zlib' could not be resolved and referenced by '@com_google_protobuf//:protobuf'

```

解决方案： 打开报错的文件 `/home/guoyt/serving/.cache/_bazel_guoyt/0456a77278cd25a6cabf8d2d5a95c9b3/external/com_google_protobuf/BUILD`， 然后编辑：`@zlib//:zlib` 这一行，改为 `@zlib_archive//:zlib`。 参考 [Google Git](https://chromium.googlesource.com/external/github.com/tensorflow/tensorflow/+/refs/heads/master/tensorflow/workspace.bzl)。



## 参考
[How to serve pytorch or sklearn models using tensorflow serving](https://stackoverflow.com/questions/49624799/how-to-serve-pytorch-or-sklearn-models-using-tensorflow-serving)

