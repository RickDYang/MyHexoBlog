---
title: Windows 10 & Ubuntu 双系统工作站
categories: 备忘 
tags: [操作系统, bootloader]
date: 2017-03-06
toc: true
---
这里记录安装双系统工作站的坑，以备忘。

## 笔记本选择
一直想折腾一个工作站笔记本，但是工作站级别笔记本太贵了，选来选去选了一个游戏本[HP暗影精灵 II 代PRO][1]，各种配置都不错。唯一的不足就是内存和SSD都太小，于是自己加了一个8G的内存和480G的SSD。总之游戏本是替代工作站的不错选择。
<!--more-->
找惠普的客服加硬件和重装系统，结果客服折腾了近一周都没把系统给装好，显卡驱动一直安装不上。实在忍不了，把机器要了回来，自己折腾。惠普客服安装的Windows 10家庭中文版确实问题很多，驱动安装不上，家庭内局域网访问也有问题，还有惠普的客服账户。强迫症不能忍。好在手头上还有Windows 10 Pro 的key，索性自己清理重装系统。
Nvdia的驱动在Windows 10 Pro上很容易地就装好了，应该是N卡驱动和家庭中文版的兼容问题。

## 双系统安装
Windows 10 和 Ubuntu安装，基本参考[Windows10+Ubuntu双系统安装][2]，里面介绍得很详细。但是在UEFI环境下安装还是有所区别。
- BIOS Security Boot
安装完Windows 10后，一定要把这个选项在BIOS关了，永远不要打开。一旦打开，系统又会直接自动Boot到Windows。我本来设置好了，又手贱去BIOS打开，结果回到解放前。一定关掉Security Boot，重要事情说三遍。
- Ubuntu /boot 安装
Windows 10先安装后，会自动划分出一个叫Windows Boot Manager的分区，这就是Windows的boot分区。在安装Ubuntu的时候，不用另外划分boot分区，直接把“安装启动引导器的设备”指定到Windows Boot Mananger所在的那个分区即可。这样Ubuntu会把Linux boot安装到和Windows同一个Boot分区。

## 系统引导
Ubuntu安装完之后，会在boot分区创建GRUB引导。但系统还是自动用Windows bootloader来进行引导。下面一些步骤来用替换Windows bootloader。
- Bootloader启动序列
而对于UEFI引导的机器是没法用EasyBCD把Ubuntu bootloader添加到Windows Bootloader的，所以只能用Grub来引导Windows。
这里可以用**BOOTICE**修改UEFI的“启动序列”，让GRUB优先引导。
（如果GRUB里面没有Windows的选项，可以在Linux用“Grub Customizer”把Windows加进去。Grub Customizer可以设置GRUB的背景字体等，但是还是丑！）
如果按后面用rEFInd做bootloader，可能就不需要用BOOTICE了。

## Bootloader美化
GRUB的界面还是太丑了，强迫症犯了，没法忍受。Google了一圈，GRUB的美化工具BURG似乎是个好的方案。折腾了半天，BURG在UEFI上一直设置不上。（如有谁折腾好了，请告诉我）。
后来找到了新的工具rEFInd，支持UEFI，而且主题可换！
在Ubuntu下安装[rEFInd][3]，非常方便。安装好后启动界面如下，还不错。
![rEFInd][4]
还有更漂亮的主题，[rEFInd Next][5]
![Next Theme][6]
但是Next Theme安装的时候有坑，不能安装到子目录下面。资源加载是从rEFInd目录下的相对路径加载，安装到子目录下面就没法找到相应资源。
正确的安装路径类似：/boot/EFI/refind/next-theme

至此强迫症问题得到解决。

## 蓝牙设备共享问题
蓝牙设备双系统共享是个大坑。因为蓝牙设备因为安全性的原因，在不同的系统上的配对码是不一样的，而设备同一时间只能工作在一个配对码下，每次切换系统都得重新配对，十分蛋痛。
我参考的是[这篇文章][7]。具体步骤如下：
- 先在Linux下配对蓝牙设备
- 再切换到Windows下配对蓝牙设备
- 再切换到Linux
这时，蓝牙设备是不工作的，因为设备工作在Windows的配对码，我们需要把Linux系统里面的配对码修改成一致。
- 找到Windows下蓝牙设备的配对码
在Linux环境下，进入Windows系统目录
```bash
cd Windows/System32/config/
chntpw -e SYSTEM 
cd ControlSet001\Services\BTHPORT\Parameters\Keys 
ls 
cd "ls 中显示的 value name的值" 
ls 
hex "ls 中显示的 value name的值"
```
   记录下上面步骤显示的Hex值（拍照...）
   
- 更改Linux蓝牙配对码
```bash
cd /var/lib/bluetooth/
ls # 如果有多个蓝牙设备，可在蓝牙设备管理里查看具体的设备编号
cd "类似AA:11:22:33:44:55"
ls
cd "类似DD:11:22:33:44:55"
ls
vim info
```
   用之前记录的Hex值（去掉空格）替换文件里面的key值
```bash
[LinkKey] 
Key=0xXXXXXXXXXXXXXXXX
```
   重启之后，大功告成。不过一旦蓝牙设备重置配对码，又得重新设置一次，还是十分蛋疼。

**免责声明**
不同的机器可能有不同的问题，我只保证在我的机器下是可行……


  [1]: http://www.hpstore.cn/wasd-ii-15-ax224tx-1.html
  [2]: http://www.jianshu.com/p/2eebd6ad284d
  [3]: http://www.rodsbooks.com/refind/
  [4]: http://www.rodsbooks.com/refind/refind.png
  [5]: http://sdbinwiiexe.deviantart.com/art/rEFInd-Next-Theme-407754566
  [6]: http://pre15.deviantart.net/9c17/th/pre/f/2013/290/b/d/refind_next_theme_by_sdbinwiiexe-d6qrlfq.png
  [7]: http://www.itdadao.com/articles/c15a638924p0.html
