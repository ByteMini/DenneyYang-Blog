---
author: Denney Yang
pubDatetime: 2022-11-11T19:02:28+08:00
title: 多种抓包工具联合分析app流量
featured: false
draft: false
tags:
- notes
description: 前言  最近需要经常对app进行抓包分析，有时单一的抓包工具对流量的分析不够全面。偶然间看到了一篇先知社区上的文章： https://xz.aliyun.com/t/11817，这篇文章讲述了多种工具联合对app抓包的方法，但根据作者的思路，我没能成功，最后我根据作者的思路换了一下代理顺序，最后成功...
---

## 前言

最近需要经常对app进行抓包分析，有时单一的抓包工具对流量的分析不够全面。偶然间看到了一篇先知社区上的文章： https://xz.aliyun.com/t/11817，这篇文章讲述了多种工具联合对app抓包的方法，但根据作者的思路，我没能成功，最后我根据作者的思路换了一下代理顺序，最后成功抓到了app流量，特此记录一下以便后期回顾，同时也希望帮助到有需要的朋友。

## 工具

* Burpsuite：抓包工具。[下载地址](https://portswigger.net/burp)、破解：谷歌或百度
* Charles：抓包工具。[下载地址](https://www.charlesproxy.com/latest-release/download.do)、[破解工具](https://www.zzzmode.com/mytools/charles/)
* Proxifier：流量代理工具。[下载地址](https://www.proxifier.com/)、破解：谷歌或百度
* adb：安卓调试工具，[下载地址](https://developer.android.com/studio/command-line/adb)
* kali虚拟机：用于转换证书
* Postern（非必须）：一款转发手机流量的app，是一个Android下的全局代理工具。不用这个app也行，直接在网络设置中配置代理也可以。[下载地址1](https://soft.clbug.com/soft/postern/)、[下载地址2](https://www.malavida.com/en/soft/postern/android/)
* v2rayN（非必须）：这里主要是有些app需要科学上网才可以打开，在这种场景下，最终我们需要一款科学上网工具将流量出口进行代理，选择其他喜欢的科学上网工具也是可以的。

## 配置过程

参考先知上的[这篇文章](https://xz.aliyun.com/t/11817)即可，配置过程大致一样，但不同的是我将burpsuite做为第一次代理，charles做为第二层代理。我主要记录的是遇到的难点。

## 代理结构

![](https://s2.loli.net/2022/11/12/rvcDPk93WuKyfl2.jpg)

## 主要难点

根据先知上那个作者的思路，使用charles作为第一层代理始终导致手机不能联网，这个问题我也不清楚为甚么！按理来说应该是可以的。最后我发现将charles证书导入burpsuite后，从burpsuite导出的证书其实是charles的。

本来使用charles可以直接将流量出口到外部的服务器，但是我也是服了，charles这个鬼东西就是转不出去，所以使用了Proxifier代理了Charles的流量。
