---
author: Denney Yang
pubDatetime: 2021-12-17T21:48:13+08:00
title: CDN绕过
featured: false
draft: false
tags:
- notes
description: 什么是CDN  CDN（content Delivery Network或Content Distribution network）即内容分发网络，为了加快网站的访问速度而设置的网站内容缓存节点。开启了CDN的网站可以让用户从离他们最近的缓存节点来访问网站内容，从而提高了网站的访问速度。  没有CD...
---

## 什么是CDN

CDN（content Delivery Network或Content Distribution network）即**内容分发网络**，为了加快网站的访问速度而设置的网站内容缓存节点。开启了CDN的网站可以让用户从离他们最近的缓存节点来访问网站内容，从而提高了网站的访问速度。

没有CDN的网站访问过程：

用户访问网站域名-->DNS服务器解析域名为IP-->访问目标主机

开启CDN的网站访问过程：

用户访问网站域名-->CDN节点-->真实服务器IP-->访问目标主机

## 对CDN的配置

* 加速域名：需要加速的域名，可以是部分域名，也可以是全部域名
* 加速区域：需要加速的地区，可以是国内，也可以是全球
* 加速资源：需要加速的网站资源类型，如图片、文件、视频......

## CDN的检测

### nslookup工具

windows自带，打开cmd，输入nslookup xxx.com

![QQ截图20211216234214](https://s2.loli.net/2021/12/17/ROUPybTmlqWo2fx.png)

如上图，出现多个IP,多半是开启了CDN

### 多地ping

多地ping就是从各个不同的地区去ping目标主机，若ping出来的结果不一致则多半开启CDN，若ping出的主机IP都一样，则没有开启CDN。

在线ping网站：

* https://www.17ce.com/
* https://ping.aizhan.com/
* http://ping.chinaz.com/

### 在线工具查询

可以根据一些网站来直接判断目标是否存在CDN

* https://www.ipip.net/ip.html
* https://www.cdnplanet.com/tools/cdnfinder/

## 绕过CDN查询真实IP

### 1.从子域名找真实IP

因为开始CDN是有费用的，所以企业为了解决成本开销，一般会给主域名开启CDN加速，而对于子域名不会开启CDN，那么我们就可以查找网站的子域名，然后ping子域名，从而找出真实的IP。

查找子域名的方法有很多，可以使用工具，也可以使用在线网站查询，Google hack等。

网站：https://dnsdb.io/zh-cn/

### 2.网站漏洞

如果网站存在漏洞，那么很有可能将敏感文件信息泄露，例如phpinfo敏感信息泄露、Apache status和Jboss status敏感信息泄露、网站源代码泄露、svn信息泄露信、github信息泄露等，这样我们就可以借助网站漏洞来找到真实IP。

如果网站存在ssrf、命令执行等漏洞，那么我们就可以利用漏洞来让目标主机主机连接我们，这样我们就可以拿到目标的真实IP。这个过程就像反弹shell过程中的正向连接、反向连接一样。

### 3.历史DNS记录

我们可以查询域名与IP的历史绑定记录，就算开启了CDN，但域名以前总有与真实IP绑定的时候，只要域名与真实IP进行过绑定，那我们就很可能从历史DNS记录中找到目标的真实IP。

查询DNS记录的网站：

* https://dnsdb.io/zh-cn/
* https://x.threatbook.cn/
* http://toolbar.netcraft.com/site_report?url=
* http://viewdns.info/
* http://www.17ce.com/
* https://community.riskiq.com/
* http://www.crimeflare.com/cfssl.html

### 4.从CDN厂商入手

可以从CDN本身入手，我们可以通过一些社会工程学技巧直接拿到该网站CDN控制面板的密码，这样直接登录CDN控制面板就可以看到网站的真实IP，这个方法如果成功实现了当然对找真实IP肯定是很稳的，但是一般情况下我们很难能拿到CDN登录面板的密码（当然是对于我这种菜鸡来说很难，大佬忽略）。

### 5.MX记录或邮件

现在的大多数网站都有邮箱发送的功能，无论你是在网站上注册，还是通过绑定的邮箱来找回密码，网站都活给你发送一个邮件到你指定的邮箱中，而邮箱系统多半是在网站内部，没有经过CDN加速，我们就可以利用这一点来获取网站的真实IP。如我们可以在网站上找回密码，然后网站给我们发送一个邮件，我们再查看邮件的源码，邮件源码中就可能包含真实的IP。

### 6.国外请求
这个还是利用了CDN的配置，我们从CDN的配置中知道，配置CDN时需要选择加速的区域，但是网站的加速区域不可能覆盖全球的每个区域，一般对于国内网站都是针对国内区域进行加速，正是因为有这样一个特点，我们可以使用一些"冷门"国家的网络代理来请求目标网站，这样我们就可以通过国外的DNS解析来取得目标的真是IP。

在线网站：

* https://tools.ipip.net/cdn.php
* https://get-site-ip.com/

### 7.全网扫描

这种方法就比较硬核了，直接使用工具扫描全网找出可能的真实IP，但为了提高效率可以结合信息收集，看网站的备案信息，网站的服务器所在地区等从而缩小查找的IP范围。

工具：https://github.com/Tai7sy/fuckcdn

### 8.网络空间引擎搜索法

使用zoomeye、fofa、shodan等暗黑搜索引擎来获得目标网站的历史快照信息，可以使用一些特征来搜索，这些特征有：

* 特有的http头部（如server类型、版本、cookie等信息)
* 特定keyword（如title、css、js、url等）
* 特定的IP段搜索（如fofa支持C段搜索）

### 9.待续

## 后续操作
我们要明白绕过CDN获取目标真实IP的目的是什么。当我们的目标网站开启CDN时，我们费尽心思得到目标的真实IP后是为了我们后续的操作，只有得到了目标的真实IP后我们之后对目标进行的操作才是有效的。如果没有找到目标真实IP，我们的操作只是针对缓冲节点。

我们获取目标的真实IP后，我们可以修改hosts文件，将目标网站与目标的真实IP进行绑定，这样当我们请求目标网站时就会解析到目标的真实IP。

hosts文件的位置：

windows：c:\windows\system32\drivers\etc\hosts

linux：etc/hosts

## 总结

判断目标网站是否存在CDN，以及绕过CDN获取真实IP对我们来说是十分重要的，因为绕过CDN找真实IP就像是找到目标真实的"门"在哪里，如果我们连目标的"门"都没有找到，那我们就不用进行后续步骤了，因为再进行下去也是徒劳。

以上绕过CDN的方法，我只是写了常见且自己实践过的，其实还有很多绕过CDN的方法，待后续学习或用到了再补充吧！

## 参考文档

https://zhuanlan.zhihu.com/p/33440472

https://www.cnblogs.com/blacksunny/p/5771827.html
