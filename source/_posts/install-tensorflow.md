---
title: Ubuntu 16.04 & GTX 1050的深度学习开发环境搭建
categories: 机器学习
tags: [机器学习]
date: 2017-03-27
toc: true
---

这是在Ubuntu 16.04 & GTX 1050上安装TensorFlow with GPU，从零开始打造深度学习开发环境。整个过程比较顺利，除了安装驱动折腾了好久。另外公司的网络虽然自带翻墙，但是会莫名其妙地屏蔽一些网站，搞得安装下载不能继续，还是自己翻墙比较实在。
<!--more-->

### 1. 安装NVIDIA Driver ###
 如果已经安装了，可以跳过这步。
#### 1.1. 官方版本 ####
 从[NVDIA官方网站][1]下载
 Ctrl+Alt+F1进入终端模式
 先卸载旧驱动
 ```bash
 sudo apt-get remove --purge nvidia*  
 ```
 安装下载的新驱动
 注意要有后面的参数——"-no-x-check --no-nouveau-check --no-opengl-files"
 否则Ubuntu可能会一直卡在login界面，我就被这个坑了好久。
 ```bash
 sudo chmod a+x NVIDIA-Linux-x86_64-378.13.run
 sudo ./NVIDIA-Linux-x86_64-375.26.run --no-x-check --no-nouveau-check --no-opengl-files
 sudo reboot
 ```
#### 1.2. 源安装 ####
 如果官方驱动安装有问题，可以尝试下面的源安装
 ```bash
 sudo add-apt-repository ppa:graphics-drivers/ppa
 sudo apt-get -qy update
 sudo apt-get -qy install nvidia-378
 sudo apt-get -qy install mesa-common-dev
 sudo apt-get -qy install freeglut3-dev
 sudo reboot
 ```
 我安装是nvidia-378，可以从下面的列表里面选个最新的安装
 ```bash
 apt list nvidia-*
 ```
### 2. 安装NVIDIA CUDA 8.0 ###
 参考步骤[NVIDIA CUDA Installation Document][2]
#### 2.1. 验证CUDA Capable GPU ####
 [NVIDIA 查询CUDA GPU网站][3]
 Notebook GTX 1050不在列表里面，而Desktop却在，WTF。不管了，继续安装，后来证明是可以的。
 用下面命令查看显卡类型
 ```bash
 lspci | grep -i nvidia
 ```
 如果不能正确显示显卡型号，需要更新一下PCI ids
 ```bash
 sudo update-pciids 
 ```
 然后再查一下显卡类型
#### 2.2. 其他验证 ####
 按[NVIDIA CUDA Installation Document][4]步骤，一步一步来。
#### 2.3. 下载CUDA 8.0 ####
 从[CUDA 8.0][5]选择对应版本下载，我选择的是runfile(local)版本。但是下载实在是太慢了，而且莫名其妙会失败。浏览器直接下载不行，NAS也不行，QQ旋风更不行，最后换成迅雷，飞快地下完了，赞！不放心的话，可以用CUDA Installer Chechsum验证。
#### 2.4. 安装CUDA ####
 ```bash
 sudo sh cuda_8.0.61_375.26_linux.run
 ```
 安装的时候会让你再次安装驱动，之前已经安装好了，千万要选No
 CUDA Samples的目录我放在了/usr/local/cuda-8.0/下
 细节可以参考[深度学习开发环境配置：Ubuntu1 6.04+Nvidia GTX 1080+CUDA 8.0][6]
### 3. 安装NVIDIA cuDNN5.1 ###
 安装cuDNN5.1需要注册NVIDIA会员，简单填一下基本信息即可。
 去[Nvidia官网][7]下载cuDNN安装包，选择这个：Download cuDNN v5.1 for CUDA 8.0 & cuDNN v5.1 Library for Linux
 解压安装包以后会出现cuda的目录，进入该目录
 ```bash
 cd cuda/include/   
 sudo cp cudnn.h /usr/local/cuda/include/   
 cd ../lib64   
 sudo cp lib* /usr/local/cuda/lib64/   
 sudo chmod a+r /usr/local/cuda/include/cudnn.h/usr/local/cuda/lib64/libcudnn*  
 ```
 接下来执行以下命令：
 ```bash
 cd /usr/local/cuda/lib64/   
 sudo rm -rf libcudnn.so libcudnn.so.5   
 sudo ln -s libcudnn.so.5.1.5 libcudnn.so.5   
 sudo ln -s libcudnn.so.5 libcudnn.so  
 ```
 在终端中输入以下命令进行环境变量的配置：
 ```bash
 sudo gedit /etc/profile  
 ```
 在末尾加上：
 ```bash
 PATH=/usr/local/cuda/bin:$PATH   
 export PATH
 ```
 创建链接文件
 ```bash
 sudo gedit /etc/ld.so.conf.d/cuda.conf 
 ```
 在该文件末尾加入
 ```bash
 /usr/local/cuda/lib64
 ```
 然后使用ldconfig使之生效
 ```bash
 sudo ldconfig
 ```
### 4. 验证CUDA & cuDNN ###
 进入CUDA 8.0 Samples默认安装路径
 ```bash
 sudo make all -j4  
 cd bin/x86_64/linux/release   
 ./deviceQuery
 ```
 cuDNN安装和验证是参考的是[Ubuntu16.04+GTX 1050+cuda8.0+cuDNN5.1+caffe安装详解][9]
### 5. 安装TensorFlow With GPU ###
 我直接参考的[TensorFlow官方Linux安装教程][10]，用的是推荐的Virtualenv安装。一路都非常顺利。

 至此，深度学习开发环境算是搭建完成了。

  [1]: http://www.geforce.cn/drivers
  [2]: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/
  [3]: http://developer.nvidia.com/cuda-gpus
  [4]: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/
  [5]: https://developer.nvidia.com/cuda-downloads
  [6]: https://zhuanlan.zhihu.com/p/22635699
  [7]: https://developer.nvidia.com/rdp/cudnn-download
  [8]: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#axzz4VZnqTJ2A
  [9]: http://www.linuxdiyf.com/linux/27958.html
  [10]: https://www.tensorflow.org/install/install_linux
