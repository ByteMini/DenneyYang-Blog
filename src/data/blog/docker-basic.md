---
author: Denney Yang
pubDatetime: 2021-09-04T20:38:43+08:00
title: 初识Docker
featured: False
draft: False
tags:
  - notes
description: Docker概述   Docker为什么出现  首先一款产品的生产要经历开发，上线等步骤，产品上线后又需要运维进行维护。而开发、运维的环境是不一样的，所以这就导致了当产品开发出来，准备部署到服务器上时，常常会出现不能正常运行的情况。我们需要在服务器上配置与开发一样的环境，但配置环境是费时费力的过程。...
---
## Docker概述

### Docker为什么出现

首先一款产品的生产要经历开发，上线等步骤，产品上线后又需要运维进行维护。而开发、运维的环境是不一样的，所以这就导致了当产品开发出来，准备部署到服务器上时，常常会出现不能正常运行的情况。我们需要在服务器上配置与开发一样的环境，但配置环境是费时费力的过程。当服务器数量较少时，配置环境的工作量还可以接受，但是当服务器数量较多时，配置环境的工作量将是巨大的。对于这样一个问题，我们能不能将项目及其环境一同打包，将项目和环境一同交给运维部署呢？这样当产品一旦开发完成，我们即可部署上线，而不用配置环境了。Docker的出现正是解决了上诉的问题。Docker的思想来源于集装箱，将各个项目及其环境装到一起，就像一个一个的集装箱。这样各个应用间相互隔离，互不干扰。Docker通过隔离机制，将服务器利用到极致。

为了展示Docker的方便之处，这里举个栗子：

一个app的流程是：Java-->apk-->发布到应用商店-->客户下载安装使用。

使用Docker发布项目：Java+环境-->打包项目带上环境（镜像）-->Docker仓库（相当于手机的应用商店）-->下载我们发布的镜像-->直接运行

### Docker的历史

2010年，几个搞IT的年轻人，在美国成立了一家公司**dotCloud**。做一些pass的云计算服务（LXC有关的容器技术）。他们将自己的容器化技术命名为Docker。Docker刚刚诞生没有引起行业的注意，所以dotCloud很快就活不下去了。为了活下去，他们决定开源源代码。在2013年，Docker开源了，逐渐的Docker被越来越多的人发现了，变火了，之后Docker几乎每个月都会更新一个版本。2014年4月9日，Docker1.0发布了！Docker为什么火了？因为它十分的轻巧。在容器技术出现之前，我们都是使用虚拟机。虚拟机和Docker都使用了虚拟化技术，但是虚拟机是模拟了一个真实的电脑，体积相当大，十分笨重，如果我们只想执行几条命令，那么虚拟机对我们的电脑的消耗是一种浪费。而Docker只是将需要的服务（如mysql,jdk等）进行打包在镜像中，我们只需要运行镜像即可启动相关的服务，而不必模拟出一个完整的电脑，所以Docker的消耗十分小，而且启动很快。

Docker是一个基于Go语言开发的开源项目。

官网：https://www.docker.com/

文档：https://docs.docker.com/

仓库：https://hub.docker.com/

### Docker能干什么

**虚拟机技术：**

![QQ截图20210904211129](https://i.loli.net/2021/11/30/KTpw2sjPDqZaEWz.png)
**虚拟机技术缺点:**

1. 资源占用十分多
2. 冗余步骤多
3. 启动很慢

**容器化技术：**

容器化技术不是模拟的一个完整的操作系统

![QQ截图20210904211716](https://i.loli.net/2021/11/30/ZUNGoX4AET19c2v.png)

**比较Docker和虚拟机技术的不同:**

* 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件
* 容器内的应用直接运行在宿主机的内核，容器是没有自己的内核的，也没有虚拟我们的硬件，所以就轻便了
* 每个容器间是互相隔离的，每个容器内都要一个属于自己的文件系统，互不影响

#### Docker能干嘛

**应用更快速的交付和部署**

传统：一堆帮助文档，安装程序

Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

使用了Docker之后，我们部署应用就和搭积木一样

项目打包为一个镜像，扩展服务器A，服务器B

**更简单的系统运维**

在容器化之后，我们的开发，测试环境都是高度一致的

**更高效的计算资源利用**

Docker是内核级别的虚拟化，可以在一个物理机上可以运行很多容器实例！服务器的性能可以被压榨到极致。

## Docker安装

### Docker的基本组成

![QQ截图20210904213835](https://i.loli.net/2021/11/30/ExeZjRaiYoz4HW7.png)

**镜像（image）：**

docker镜像就好比是一个模板，可以通过这个模板来创建容器服务，tomcat镜像-->run-->tomcat01容器（通过服务），通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）。

**容器（container）:**

Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来创建的。

这里有启动、停止、删除等基本命令

目前就可以把这个容器理解为就是一个简易的linux系统

**仓库（respository）:**

仓库就是存放镜像的地方

仓库分为公有仓库和私有仓库

Docker Hub（默认是国外的）

阿里云等都有容器服务器（配置镜像加速)

### 安装Docker

**环境准备**

1. 会一点点的linux基础
2. 一个linux系统（我使用的是CentOS 7）
3. 如果是远程服务器还需要一个远程连接工具（如xshell）

**环境查看**

```shell
# 查看系统内核，系统内核需要3.10以上
uname -r

# 查看系统版本
cat /etc/os-release
```
**开始安装**

参考官方帮助文档：https://docs.docker.com/engine/install/

```shell
# 1.卸载旧版本Docker
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2.需要的安装包
yum install -y yum-utils

# 3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo   # 默认是国外的仓库
   
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo    # 这里是设置到阿里云的仓库
    
# 更新软件包索引
yum makecache fast 

# 4.安装Docker相关的内容 docker-ce:社区版  docker-ee:企业版
yum install docker-ce docker-ce-cli containerd.io

# 5.启动Docker
systemctl start docker

# 6.测试Docker是否安装成功
docker version

# 7.运行hello-world镜像，验证docker是否安装正确
docker run hello-world

# 8.查看一下下载的hello-world镜像
docker images
```

了解一下如何卸载docker:

```shell
# 1.卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 2.删除资源
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

# /var/lib/docker docker默认的工作路径
```

### 阿里云镜像加速

为了加快从仓库获取镜像的速度，我们还需要配置一下阿里云镜像加速

1. 登录自己的阿里云控制台，找到容器镜像服务
![QQ截图20210905173210](https://i.loli.net/2021/11/30/vaDRS6MonUdmGFT.png)
2. 在容器镜像服务中找到镜像加速器
![QQ截图20210905173402](https://i.loli.net/2021/11/30/YFPg2q98V6uS5O3.png)
3. 配置镜像加速器

```shell
sudo mkdir -p /etc/docker   

sudo tee /etc/docker/daemon.json <<-'EOF'
{
   "registry-mirrors": ["https://l5b3ez5n.mirror.aliyuncs.com"]
}
EOF
   
sudo systemctl daemon-reload
   
sudo systemctl restart docker
```
## 回顾HelloWorld运行流程

![QQ截图20210905173712](https://i.loli.net/2021/11/30/HP5Q9Oy6RJ8ICqu.png)

## 底层原理

### Docker是怎么工作的？

Docker是一个Client - Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问!
DockerServer接收到Docker-Client的指令，就会执行这个命令!

![QQ截图20210905174219](https://i.loli.net/2021/11/30/2zUSvMyk3upHXEW.png)

### Docker为什么比VM快

Docker有着比虚拟机更少的抽象层。

Docker利用的是宿主机的内核，vm需要是Guest OS。

![QQ截图20210905175218](https://i.loli.net/2021/11/30/7pZ6vOygqAtaCiG.png)

所以说，新建一个容器的时候，docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载Guest OS，分钟级别的，而docker是利用宿主机的操作系统，省略了这个复杂的过程，秒级!
之后学习完毕所有的命令，再回过头来看这段理论，就会很清晰!
