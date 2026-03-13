---
author: Denney Yang
pubDatetime: 2022-02-06T21:53:10+08:00
title: VirtualBox导入vulnhub靶机各种报错
featured: False
draft: False
tags:
  - notes
description: VirtualBox没有安装扩展包报错  ![QQ截图20220206140308](https://s2.loli.net/2022/02/06/potSrD78QI9HNgE.png)   出现问题的原因  VirtualBox没有安装扩展包   解决方法  到官方（https://www.vi...
---
## VirtualBox没有安装扩展包报错

![QQ截图20220206140308](https://s2.loli.net/2022/02/06/potSrD78QI9HNgE.png)

### 出现问题的原因

VirtualBox没有安装扩展包

### 解决方法

到官方（`https://www.virtualbox.org/`）下载对应版本的扩展安装包
![QQ截图20220206140913](https://s2.loli.net/2022/02/06/p8AsvlLmio2Cugj.png)

## 导入靶机点击正常启动报错

![QQ截图20220206141111](https://s2.loli.net/2022/02/06/bGq79ZuAdz1PxNo.png)

### 出现问题的原因

没有找到原因，从给出的问题描述中是说在启动的过程中发生了意想不到的错误，搞不懂哪里有问题...

### 解决方法

虽然不能点击正常启动，但是我们可以尝试点击其他两种启动模式啊（无界面启动，分离式启动）。
![QQ截图20220206141709](https://s2.loli.net/2022/02/06/PYe4xJ9sfEmMpT6.png)

我用的无界面启动，当虚拟机启动起来时，再点击显示，就可以正常使用了。
![QQ截图20220206141857](https://s2.loli.net/2022/02/06/aT71ysdt3bfhEZq.png)

## 启动时报错'COM error'

![QQ截图20220206141942](https://s2.loli.net/2022/02/06/YhKkvUixQNGJdfP.png)

### 出现问题的原因

在网上寻找了很久，还是没有找到出现这个问题的原因，各种尝试还是不能解决，希望知道这个问题的大佬能告诉我这个菜鸡！

### 解决办法

经过各种尝试，终于发现出现这个问题好像和虚拟机的网络设置有关，最后发现将虚拟机的网络改为NAT或NAT网络可以正常启动虚拟机。
![QQ截图20220206142643](https://s2.loli.net/2022/02/06/D9enfHYFPMBjumy.png)

经过测试发现，将网络设置为NAT，虚拟机只能和物理机通讯，将网络设置为NAT网络是可以让虚拟机之间相互通讯。当使用NAT网络时需要在全局设定中为NAT网络分配网段。
![QQ截图20220206143111](https://s2.loli.net/2022/02/06/uV6QTg19o4q3MPh.png)

最后在VirtualBox中的虚拟机之间终于可以相互通讯了，本来是打算只让VirtualBox里单纯的跑靶机，在vmware中使用虚拟机对靶机进行攻击，没想到VirtualBox中导入靶机报出这么多问题。

## 参考文档

https://blog.csdn.net/super828/article/details/117151159

https://blog.csdn.net/wy_bk/article/details/78926646
