---
layout:     post
title:      Keras GPU运算环境搭建
category: blog
description: 使用Keras训练神经网络时，使用GPU训练比使用CPU训练，可以节约更多的时间。本文简单的说明如何搭建GPU的神经网络训练环境，加快神经网络的训练。
---

## 0X00 缘起

在工作中，意外发现这篇blog，[Using the Power of Deep Learning for Cyber Security (Part 1)](https://www.analyticsvidhya.com/blog/2018/07/using-power-deep-learning-cyber-security/)，出于“好奇害死猫”，考虑实现一下，看看效果如何？

在实现了Demo后，发现CPU计算较慢，同时手边有一个笔记本显卡是GTX 1050Ti，在查询了部分显卡资料后，可以确定1050Ti是适用于“随便玩玩的深度学习菜鸟”的，例如“我”。因此，本着“不作不死”的心态，开始折腾起GPU了。

## 0X01 环境

**操作系统：** Win10(Home)，系统更新到最新版。
**显卡：**  GTX1050Ti OC。
**Python：** Python3.6。
**深度学习第三库：** Keras，Tensorflow（GPU版）。

## 0X02 步骤

安装步骤如下：

1. 安装Python3.6，一路双击下去即可；记得设置环境变量，便于使用pip工具。
2. 安装Visual Studio 2017 社区版；注意，不要使用最新的Visual Studio 2019版本。
3. 安装CUDA；注意，保持默认设置就好。
4. 安装cuDNN；
5. 安装Tenserflow GPU版本：`pip install tensorflow-gpu`。
6. 安装Keras：`pip install keras`。

说明:
   1. cuDNN需要注册 NVIDIA 的开发者账号；
   2. cuDNN的安装是将下载的文件，copy到CUDA的安装目录相同的目录名下；
   3. CUDA的版本建议选择10.0即可，最新的10.1版本，Tensorflow-gpu并不支持；
   4. 由于Keras的依赖包众多，可以先用`pip`工具安装一些工具，例如：numpy等；非天朝局域网的，可忽略这条；

## 0x03 测试

使用了 [从Keras开始掌握深度学习-5 优化你的模型](https://www.jianshu.com/p/6e50d6136892)中的代码测试后，可以正确运行。此外，运算时间也大大降低。Mac book pro需要十几个小时的计算时间，此时只需要不到两小时的计算时间，坏处是计算完成，电脑温度挺高。总体而言，这个折腾还是挺值得的。可以作为菜鸟，继续欢快的折腾神经网络了。 :-) :-)

## 0X04 小结

从折腾的过程来看，主要耗费的时间就是版本的回退，重新安装的时间。刚开始选用最新的CUDA 10.1，然后回退到CUDA 10，再回退Visual Studio 版本，由于CUDA、Visual Studio 的体积庞大，以及局域网的奇葩速度，耗费了整整一天的时间。在此，吐槽局域网，访问国际站点真是慢、真慢、慢。


## 参考

1. [Using the Power of Deep Learning for Cyber Security (Part 1)](https://www.analyticsvidhya.com/blog/2018/07/using-power-deep-learning-cyber-security/)
2. [深度学习显卡选型指南:关于GPU选择的一般建议](http://m.elecfans.com/article/737945.html)
3. [CUDA Toolkit Download](https://developer.nvidia.com/cuda-downloads)
4. [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)
5. [从Keras开始掌握深度学习-5 优化你的模型](https://www.jianshu.com/p/6e50d6136892)