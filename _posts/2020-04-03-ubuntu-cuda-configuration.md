---
layout:     post
title:    Ubuntu 18.04 LTS CUDA Configuration
category: blog
description: Ubuntu 18.04 LTS配置CUDA环境，原以为会很容易，结果事实不是那么回事。这里作一个记录。
---

## 0x00 Introduction

以前配置过Win10的CUDA环境，认为Ubuntu的系统上会更容易配置。呵呵，结果惨兮兮。这里记录一下配置方法，以及配置时遇到的坑。

## 0x01 Environment

首先需要看看CUDA支持的系统版本，内核版本，gcc版本，glic版本。一定要仔细看完，否则后续出现的莫名其妙的错误，可以让你发疯。

**一定要查看Nvidia官网的安装说明，注意其中的匹配版本。一定要看，一定要看，一定要看。重要的话重复三遍。**

首先确定要安装的OS是: **Ubuntu 18.04.3 LTS**，注意不是18.04.4 LTS。其次要注意CUDA要求的内核版本是 **5.0.0**，gcc版本是 **7.4.0**，glibc版本是 **2.27**。

注意⚠️：注意上述的版本号。不对应的话，呵呵。 :-) :-)

## 0x02 Downgrade Kernel

系统安装完成后，查看内核的版本号：

```bash
$ uname -r
5.3.0-45-generic
```

因此首先需要降级系统内核，查看系统可用的Kernel：

```bash
$ dpkg --get-selections | grep linux-image
```

Install OS Kernel：

```bash
$ sudo apt install linux-image-5.0.0-23-generic linux-headers-5.0.0-23-generic
```

卸载不需要的Kernel：

```bash
$ sudo apt remove linux-image-5.3.0-45-generic linux-headers-5.3.0-45-generic
```

Update grub：

```bash
$ sudo update-grub
```

重启系统，确认内核版本符合。

```bash
$ uname -r
5.0.0-23-generic
```

## 0x03 downgrade gcc

Ubuntu gcc version：

```bash
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc

... ...

Thread model: posix
gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)
```

注意：自带的gcc版本是7.5.0，因此需要downgrade gcc。

由于Ubuntu官方的软件库不支持低版本的gcc在线安装，因此需要手工下载gcc-7.4.0.tar.gz，编译安装：

```
$ cd gcc-7.4.0/

# 下载编译的依赖软件
$ ./contrib/download_prerequisites

# create build dir
$ mkdir gcc-build

$ cd gcc-build

# build and install 
$ ../configure --enable-checking=release
$ make -j8
$ sudo make install
```

验证gcc版本：

```bash
$ gcc -v

... ...

Thread model: posix
gcc version 7.4.0 (GCC)
```

gcc版本为7.4.0，满足要求。


查看系统glibc版本：

```bash
$ getconf GNU_LIBC_VERSION
glibc 2.27
```

这样gcc，glic的版本均符合Nvidia的官方推荐。


附：C++动态库版本查看：

```bash
$ strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 |grep GLIBCXX
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBCXX_3.4.25
GLIBCXX_DEBUG_MESSAGE_LENGTH
```

查看glibc版本：

```bash
$ strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC_
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_2.18
GLIBC_2.22
GLIBC_2.23
GLIBC_2.24
GLIBC_2.25
GLIBC_2.26
GLIBC_2.27
GLIBC_PRIVATE
```

## 0x04 Install CUDA

这里使用的是CUDA-10.1的版本，与之搭配的cuDNN版本为7.6。我们选择deb[local]的方式来安装，官网提供的安装命令如下：

```bash
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
$ sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb
$ sudo apt-key add /var/cuda-repo-10-1-local-10.1.243-418.87.00/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get -y install cuda
```

安装完成后，需要添加环境变量等如下：

```bash
export PATH=/usr/local/cuda-10.1/bin:/usr/local/cuda-10.1/NsightCompute-2019.1${PATH:+:${PATH}}

export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

验证CUDA：

```bash
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243
```

Install cuDNN，下载cuDNN的包（需注册），解压得到内部的文件：

```bash
$ sudo cp libcudnn_static.a libcudnn.so.7.6.5 /usr/local/cuda-10.1/lib64
$ sudo cp ../include/cudnn.h /usr/local/cuda-10.1/include

$ cd /usr/local/cuda-10.1/lib64
$ sudo ln -s libcudnn.so.7.6.5 libcudnn.so.7
$ sudo ln -s libcudnn.so.7 libcudnn.so

$ sudo chmod a+r libcudnn.so.7.6.5 libcudnn.so.7 libcudnn.so
$ sudo chmod a+r cudnn.h
```

## 0x05 Install Tensorflow

### 0x0501 Install pip

下载官网的`get-pip.py`文件：

```bash
$ sudo python3 get-pip.py
```

pip3安装时报错：

```bash

... ...

launchpadlib 1.10.6 requires testresources, which is not installed.
```

解决：

```bash
$ sudo apt install python3-widgetsnbextension
$ sudo apt install python3-testresources
```

### 0x0502 Install Tensorflow

从Tensorflow的官网来看，新版本的Tensorflow不再区分CPU和GPU版本；1.15以前的版本还是区分的。环境不实用老版本的Tensorflow，因此，不考虑CPU和GPU版本区分问题。

Tensorflow的安装以及常用的包安装：

```bash
pip3 install numpy
pip3 install pandas
pip3 install scipy

pip3 install tensorflow
```


## 0x06 Troubleshooting

执行一个测试代码，报错信息如下：

```bash
2020-04-07 11:07:07.697146: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library libcublas.so.10
2020-04-07 11:07:07.823322: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library libcudnn.so.7
2020-04-07 11:07:08.203859: E tensorflow/stream_executor/cuda/cuda_dnn.cc:329] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2020-04-07 11:07:08.207765: E tensorflow/stream_executor/cuda/cuda_dnn.cc:329] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2020-04-07 11:07:08.207819: W tensorflow/core/common_runtime/base_collective_executor.cc:217] BaseCollectiveExecutor::StartAbort Unknown: Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.
```

简化一下：

```bash
UnknownError: Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above. [Op:Conv2D]
```

解决办法：添加下面两行

```python
import os
os.environ['CUDA_VISIBLE_DEVICES'] = '/gpu:0'
```

注意：上述办法未解决问题。

使用google提供的[TF Keras: Basic classification: Classify images of clothing](https://tensorflow.google.cn/tutorials/keras/classification)的例子测试cuda以及cudnn的配置。测试结果表明：测试环境配置是正确的。因此后续需要考虑的问题是如何修改代码，来适配新版本的Tensorflow。

测试demo：

```python
#! /usr/bin/env python3
# -*- coding: utf-8 -*-

import tensorflow as tf
from tensorflow import keras
import numpy as np

def keras_test():
    gpu_list = tf.config.experimental.list_physical_devices(device_type='GPU')
    tf.config.experimental.set_visible_devices(devices=gpu_list[0],device_type='GPU')
    tf.config.experimental.set_memory_growth(gpu_list[0], True)

    fashion_mnist = keras.datasets.fashion_mnist
    (train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()
    class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat',\
                   'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']
    print(train_images.shape)

    train_images = train_images / 255.0
    test_images = test_images / 255.0
    model = keras.Sequential([keras.layers.Flatten(input_shape=(28, 28)),\
                              keras.layers.Dense(128, activation='relu'),\
                              keras.layers.Dense(10)])
    model.compile(optimizer='adam',\
                  loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),\
                  metrics=['accuracy'])
    model.fit(train_images, train_labels, epochs=10)
    test_loss, test_acc = model.evaluate(test_images, test_labels, verbose=2)
    print('\nTest accuracy:', test_acc)

    return

if __name__ == '__main__':
    print("hello python")
    print(tf.__version__)
    keras_test()
```

## Reference

1. [Tensorflow GPU 支持](https://tensorflow.google.cn/install/gpu)
2. [cuDNN Archive](https://developer.nvidia.com/rdp/cudnn-archive)
3. [cuda-10.2: NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions)
4. [cuda-10.1: NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/archive/10.1/cuda-installation-guide-linux/index.html)
5. [CUDA Toolkit 10.1 original Archive](https://developer.nvidia.com/cuda-10.1-download-archive-base?target_os=Linux)
6. [Ubuntu更改内核](https://blog.csdn.net/qq_40930088/article/details/103392342)
7. [Ubuntu系统Linux内核升级、降级](https://blog.csdn.net/andyL_05/article/details/89877063)
8. [Ubuntu安装CUDA Toolkit 10.2](https://blog.csdn.net/qq_40930088/article/details/103389244)
9. [tensorflow2.0卷积报错:Failed to get convolution algorithm](https://www.jianshu.com/p/e13aa15f35da)
10. [TF Keras: Basic classification: Classify images of clothing](https://tensorflow.google.cn/tutorials/keras/classification)
11. [CUDA Toolkit 10.1 update2 Archive](https://developer.nvidia.com/cuda-10.1-download-archive-update2)