---
author: Denney Yang
pubDatetime: 2022-02-07T20:20:44+08:00
title: Linux中使用Qv2ray
featured: false
draft: false
tags:
- notes
description: 前言  最近有在Linux上使用机场工具的需求，但在网上找了一圈发现Linux上可用的机场工具相当少，大部分的机场工具都是在windows和mac平台上使用的，最后还是找到了一个Linux上可用的机场工具——Qv2ray。但是这个工具在2021.8.17已经停止维护了，但以前的版本还是可用的，有总比...
---

## 前言

最近有在Linux上使用机场工具的需求，但在网上找了一圈发现Linux上可用的机场工具相当少，大部分的机场工具都是在windows和mac平台上使用的，最后还是找到了一个Linux上可用的机场工具——Qv2ray。但是这个工具在2021.8.17已经停止维护了，但以前的版本还是可用的，有总比没有好，凑合着用吧。

## 下载Qv2ray客户端文件

下载地址： https://github.com/Qv2ray/Qv2ray/releases

我下载的版本是2.7.0，是Qv2ray目前的最终版本

![QQ截图20220207115051](https://s2.loli.net/2022/02/07/WmMwjpFZ56EsKA7.png)

## 下载v2ray核心文件

上面下载的Qv2ray只是一个“空壳”，如果要实现核心功能，还要下载v2ray核心文件 

下载地址： https://github.com/v2fly/v2ray-core/releases

我这里下载的是4.44.0版本

![QQ截图20220207115717](https://s2.loli.net/2022/02/07/wQxAPYVTbaqmn3S.png)

## 将下载的Qv2ray和v2ray拖入要安装的机器里

我使用的是kali2021.4

![QQ截图20220207121742](https://s2.loli.net/2022/02/07/FIsledkMD35LhBu.png)

赋予Qv2ray执行权限

`sudo chmod +x ./Qv2ray-v2.7.0-linux-x64.AppImage`

打开Qv2ray客户端

`sudo ./Qv2ray-v2.7.0-linux-x64.AppImage`

![QQ截图20220207122130](https://s2.loli.net/2022/02/07/H3LpnSfPXkOioBb.png)

## 配置Qv2ray

解压v2ray压缩包，解压到当前文件目录中的v2ray-linux目录

`sudo unzip -d ./v2ray-linux ./v2ray-linux-64.zip `

将Qv2ray调整为中文

![QQ截图20220207123720](https://s2.loli.net/2022/02/07/6Urlp5WoZGgMFv4.png)

配置v2ray核心执行文件和资源目录

![QQ截图20220207124102](https://s2.loli.net/2022/02/07/OXvYCdI4msMa6kg.png)

## 导入订阅更新节点

把自己的订阅导入到Qv2ray

![QQ截图20220207125106](https://s2.loli.net/2022/02/07/AF3tPf4daX6Bp1H.png)

连接一个节点（**这里感谢工具大师提供的订阅节点**）

![QQ截图20220207125349](https://s2.loli.net/2022/02/07/HP3w4xY5gEUclBX.png)

## 设置代理

![QQ截图20220207132713](https://s2.loli.net/2022/02/07/A7cL3pN2qH1v5D8.png)

设置入站规则

![QQ截图20220207133027](https://s2.loli.net/2022/02/07/S8dHMiZbsOuGhAC.png)

到浏览器设置代理

![QQ截图20220207133205](https://s2.loli.net/2022/02/07/PMmXp72I6DksCc8.png)

访问youtube试试

![QQ截图20220207133243](https://s2.loli.net/2022/02/07/bTPCfisuDVXWL9K.png)

大功告成！！！

## 参考文档

`https://www.zsxcool.com/7137.html`

`https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%BD%BF%E7%94%A8%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%90%86`

`https://sites.google.com/view/honven/%E9%A6%96%E9%A1%B5/qv2ray%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B`
