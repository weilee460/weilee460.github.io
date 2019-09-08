---
layout:     post
title:      Nvida Jetson Nano Start
category: blog
description: Nvida Jetson Nano 是类似于raspberry的卡片机，不同的是支持gpu运算，便于机器学习和深度学习算法的测试、使用。
---

## 0x00 OS Install

准备一张sd卡，下载nano系统镜像，下载链接见[3]，下载镜像烧写软件，官方推荐Etcher，见[5]。

官方上手指南见[4]。

## 0x01 Environment Configuration

官方的系统已经将cuda，cudnn等软件库集成在系统中，因此只需要简单的配置即可使用。

1. 首先添加环境变量（此系统比较怪异，居然不是常见的`.bash_profile`文件）：

    ```
    vim .profile
    #下面三行添加到文件的末尾
    export PATH=/usr/local/cuda-10.0/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
    export CUDA_HOME=$CUDA_HOME:/usr/local/cuda-10.0
    ```

2. 测试是否配置正确：

    ```
    $ nvcc --version
    #正确的显示如下
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2019 NVIDIA Corporation
    Built on Mon_Mar_11_22:13:24_CDT_2019
    Cuda compilation tools, release 10.0, V10.0.326
    ```

3. 测试cudnn：

    ```
    # 官方自带的测试用例
    cd /usr/src/cudnn_samples_v7/mnistCUDNN/
    make
    ./mnistCUDNN
    ```

    测试通过的标志是一些列的`pass`，例如：

    ```
    Resulting weights from Softmax:
    0.0000000 0.0000008 0.0000000 0.0000002 0.0000000 1.0000000 0.0000154 0.0000000 0.0000012 0.0000006

    Result of classification: 1 3 5

    Test passed!
    ```

4. 配置tensorflow以及keras环境：

    ```
    #安装pip3
    sudo apt install python3-pip python3-dev
    #安装所需的软件库
    sudo apt install gfortran libopenblas-dev liblapack-dev libatlas-base-dev libhdf5-serial-dev hdf5-tools zlib1g-dev libprotobuf-dev libleveldb-dev libsnappy-dev  protobuf-compiler
    #不解释
    sudo pip3 install numpy pandas
    #安装scipy，需要先安装上面的软件库，否则会报错：
    sudo pip3 install scipy
    #Install Cython
    sudo pip3 install cython
	 #需要先安装cython，否则安装sklearn时会报错
	 sudo pip3 install sklearn
	 #安装jupyter：
	 sudo pip3 install jupyter
	 #
	 sudo pip3 install grpcio absl-py py-cpuinfo psutil portpicker six mock requests gast h5py astor termcolor protobuf keras-applications keras-preprocessing wrapt google-pasta setuptools testresources
	 #
	 sudo pip3 install tensorflow-estimator
	 
	 sudo pip3 install tensorboard
    
    #先装上面两个，再安装tensorflow gpu版本
	sudo -H pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 tensorflow-gpu
    ```
    
5. 测试tensorflow是否安装正确：

	```
	#不报错
	$ python3
	>>> import tensorflow
	```

6. 安装keras：

	```
	sudo pip3 install keras
	```

7. 测试keras：

	使用参考[7]中的gpu的神经网络训练代码来测试即可。


## Reference

1. [Jetson Nano配置与使用（5）cuda测试及tensorflow gpu安装](https://www.jianshu.com/p/9decb97db6bc)
2. [deep learning frameworks doc](https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html)
3. [Jetson Download Center](https://developer.nvidia.com/embedded/downl9oads)
4. [Getting Started With Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit)
5. [Etcher](https://www.balena.io/etcher/)
6. [从Keras开始掌握深度学习-2 准备你的数据集](https://www.jianshu.com/p/cf98e019b519)
7. [从Keras开始掌握深度学习-5 优化你的模型](https://www.jianshu.com/p/6e50d6136892)
