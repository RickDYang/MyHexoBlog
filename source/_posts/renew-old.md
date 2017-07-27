---
title: 旧瓶装新酒 - 训练机搭建
mathjax: true
categories: 备忘 
tags: [备忘, 系统]
date: 2017-07-27
---

一直就想搞台测试机，一来之前的机器显存不足，二来训练的时候太占用机器资源，三来几年前攒的台式机性能，还可以加块显卡就能用。旅游回来后，查了一下GTX 1070，发现有货价格还是略高，挣扎了2秒后，果断下单，京东当天就送到了。
<!-- more -->
结果拆开一看很大，很勉强地装到了老机箱，而且点不亮，电源带不动……马上又去定了机箱和电源，顺便加了块内存，幸好最后没有连主板和CPU一起换了。
硬件方面的安装一切都很顺利，之前有块SSD，准备拿来转Ubuntu。
软件方面就没有那么顺利了……
## 安装Ubuntu ##
按我之前写的[Windows 10 & Ubuntu 双系统工作站][1]，Linux的Bootloader可以和Win10安装到同一个EFI分区。可是问题来，老机器的系统是从Win7升级到Win10的，当时硬盘是MBR，并没有转成GPT，所以没有EFI分区。当然DiskGenius可以无损地把MBR转成GPT，并把Boot区转成EFI。而我适用了另一种相对安全的方案。不过在做这些事情前，请备份你的重要数据！！！
Windows 10新出了功能，可以无损转GPT，但是在创新者更新里才加入。于是
### 安装Windows 10 创新者更新 ###
我还没有被推送相关更新，只能手动安装。
到[微软官网][2]上点击“立即更新”下载“微软易升程序”，然后直接在线升级
### mbr2gpt工具 ###
重启进入PE模式，先进行验证
```
mbr2gpt /validate
```
验证通过才说明你的硬盘能转成GPT，默认是第一块硬盘，也可以用指定硬盘
```
mbr2gpt /validate /disk:0
```
验证通过后就可以转换了
```
mbr2gpt /convert /disk:0
```
详情可以参考[如何将MBR磁盘转换为GPT][3]

Ubuntu的Bootloader就可以顺利安装到同一个EFI分区了。
但是装rEFInd引导的时候很伤，Win10一从rEFind引导就黑屏，而从Grub引导或者直接启动就一切安好。Google了半天也没有什么解决方案，只好放弃rEFInd引导，而用Grub。

### Ubuntu启动黑屏 ###
Ubuntu一旦更新系统就黑屏，主要是Ubuntu对N卡支持的问题。
在启动进入grub后，按e键进入引导临时编辑模式，按键盘左右箭头移动光标，找到 quiet splash，在后面添加nomodeset，注意中间有个空格，然后按F10启动系统，就可以进入桌面了。当然这是临时方案，我们需要修改grub引导
进去Ubuntu桌面后，编辑/etc/default/grub，修改
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```
为
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
```
再更新Grub，重启电脑即可
```
sudo update-grub
```
### 显卡驱动安装 ###
在官网上下载的驱动安装一直都不顺利，用源安装最保险
```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get -qy update
sudo apt-get -qy install nvidia-381
sudo reboot
```
## 安装机器学习环境 ##
安装CUDA & cuDNN5.1，具体参考[我之前的博文][4]
我以后决定基于Keras做开发了，直接安装Anaconda和Keras即可
### 安装Anaconda ###
直接上[官网][5]下载安装即可
### 安装Keras ###
运行下列命令安装keras即可，包括相关的依赖也会一起安装上，比如tensorflow/theano/numpy
```
conda install keras
```
注意上面安装的tensorflow是非GPU版，所以还需要重新安装tensorflow-gpu版
```
conda install tensorflow-gpu
```

## 其他 ##
### Ubuntu外接显示 ###
笔记本外接显示器需要用xrandr来修改配置，否则会重新屏幕拉长，鼠标位置不正确等一系列的问题。
下面命令是将HDMI输入和笔记本自带显示屏的输入一致，具体设备号可以用xrandr直接参看。
```
xrandr --output HDMI-0 --auto --same-as eDP-1-1
```
不过这个也是临时命令，需要让它在开机运行。
在etc下新建一个可执行的脚本，再添加到开机运行即可。

## End ##
有台训练机真的不错，在训练机训练的时候，可以在笔记本上做其他事情了，比如写博客……


  [1]: http://rickdyang.me/2017-03/win-linux-setup/
  [2]: https://www.microsoft.com/zh-cn/software-download/windows10
  [3]: http://www.win10zhijia.com/news/22634.html
  [4]: http://rickdyang.me/2017-03/install-tensorflow/
  [5]: https://www.continuum.io/downloads