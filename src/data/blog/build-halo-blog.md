---
author: Denney Yang
pubDatetime: 2021-11-29T22:00:23+08:00
title: 搭建halo博客
featured: false
draft: false
tags:
- notes
description: 前言  博客搬家啦！！！★,°:.☆(￣▽￣)/$:.°★ 。经过两天的折腾终于把新博客搭建好了，新博客使用halo博客系统，同时在服务器上使用Docker进行部署，再使用nginx做反向代理。最开始是在Debian系统上搭建的，但后来因为一些原因又将服务器系统换成了CentOS，本着日常记坑的优良...
---

## 前言

博客搬家啦！！！*★,°*:.☆(￣▽￣)/$:*.°★* 。经过两天的折腾终于把新博客搭建好了，新博客使用halo博客系统，同时在服务器上使用Docker进行部署，再使用nginx做反向代理。最开始是在Debian系统上搭建的，但后来因为一些原因又将服务器系统换成了CentOS，本着日常记坑的优良做事原则，肯定得把搭建过程中遇到的坑点给记录一下，方便自己日后回顾，同时也希望能给正在搭建halo博客的朋友们一点参考。

## Debian系统上搭建

### 安装Docker

参考文档： https://docs.docker.com/engine/install/debian/#upgrade-docker-engine

```shell
# 卸载旧版本docker
sudo apt-get remove docker docker-engine docker.io containerd runc

# 设置仓库
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
# 安装docker
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

# 安装完成后，查看docker版本
docker version

```

### 拉取halo的镜像

参考文档： https://docs.halo.run/getting-started/install/docker

```shell
# 创建工作目录
mkdir ~/.halo && cd ~/.halo

# 下载示例配置文件到工作目录
wget https://dl.halo.run/config/application-template.yaml -O ./application.yaml

# 编辑配置文件，配置数据库或者端口等，如需配置请参考参考配置
# 如果不需要更改配置文件，这一步可以跳过
vim application.yaml

# 拉取Halo 镜像
docker pull halohub/halo:1.4.13

# 创建容器
docker run -it -d --name halo -p 8090:8090 -v ~/.halo:/root/.halo --restart=unless-stopped halohub/halo:1.4.13

# 容器创建完成后，进行测试
# 打开 http://ip:端口号（默认8090） 即可看到安装引导界面。

```

### 安装nginx

`sudo apt-get install nginx`

配置反向代理

`vim /etc/nginx/nginx.conf`

```shell
upstream halo {
  server 127.0.0.1:8090;
}
server {
  listen 80;
  listen [::]:80;
  server_name www.yourdomain.com（这里改为你的域名：www.xxx.com xxx.com）;
  client_max_body_size 1024m;
  location / {
    proxy_pass http://halo;
    proxy_set_header HOST $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

修改完配置文件后需要需要重启nginx

```shell
# 指定nginx的配置文件,并检查语法有无错误
nginx -t -c /etc/nginx/nginx.conf

# 重启nginx服务器
nginx -s reload
```

debian要在ufw防火墙里开放80端口后，浏览器才可以访问网站

参考文档： https://blog.csdn.net/ddu32445/article/details/102377493

```shell
# 防火墙允许nginx full
ufw allow 'Nginx Full'

# 打开防火墙后，尝试使用域名访问一下，看是否能够到达博客安装导航页
```

### 安装ssl证书

参考文档： https://www.howtoing.com/install-free-ssl-certificate-for-nginx-on-debian-10

在Debian 10中安装Certbot

```shell
# 更新Debian系统包
sudo apt update -y

# 安装python3-certbot-nginx包所需的依赖项
sudo apt install python3-acme python3-certbot python3-mock python3-openssl python3-pkg-resources python3-pyparsing python3-zope.interface

# 安装python3-certbot-nginx包
sudo apt install python3-certbot-nginx

```

验证Nginx服务器块配置

为了让ssl能自动部署到你的nginx上，需要保证以下两点：

* 在/etc/nginx/sites-available/目录下有一个文件名为你配置的域名的文件，如：/etc/nginx/sites-available/howtoing.com

```shell
# 如果没有则创建一个文件
touch /etc/nginx/sites-available/howtoing.com
```

* 确保/etc/nginx/nginx.conf文件中的server_name指令与您的域名相对应

```shell
server_name howtoing.com www.howtoing.com;
```

如果以上两点没有问题后，则最后检查一下nginx配置：

```shell
sudo nginx -t
```

部署加密SSL证书

```shell
sudo certbot --nginx -d domain-name  -d www.domain-name.com
```

根据提示进行配置

* 填写邮箱

![QQ截图20211129142216](https://i.loli.net/2021/12/03/FCXn7P1g5Skpy8w.png)

* 同意服务条款

![QQ截图20211129142635](https://i.loli.net/2021/12/03/P7V23W5zDQeknTv.png)

* 接受许可协议(Certbot将继续征求您的同意，使用您的电子邮件向您发送有关EFF最新发展的通知)

![QQ截图20211129142722](https://i.loli.net/2021/12/03/eLBhA7j86yqgiSK.png)

* 将常用HTTP流量重定向到HTTPS

![QQ截图20211129143114](https://i.loli.net/2021/12/03/dTomv7MnExIfhSr.png)

* 看到以下界面则表示部署ssl成功

![QQ截图20211129143257](https://i.loli.net/2021/12/03/mOT1ZxS7b2zBotd.png)

我们打开浏览器，浏览一下自己的网站，看网站的url处有无一把小锁，出现一把小锁则表明ssl证书生效。

## CentOS系统上搭建

### 安装Docker

参考官方帮助文档： https://docs.docker.com/engine/install/centos/

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
    
# 更新软件包索引
yum makecache fast 

# 4.安装Docker相关的内容 docker-ce:社区版  docker-ee:企业版
yum install docker-ce docker-ce-cli containerd.io

# 5.启动Docker
systemctl start docker

# 6.测试Docker是否安装成功
docker version
```

### 拉去halo的镜像

参考文档： https://docs.halo.run/getting-started/install/docker

```shell
# 创建工作目录
mkdir ~/.halo && cd ~/.halo

# 下载示例配置文件到工作目录
wget https://dl.halo.run/config/application-template.yaml -O ./application.yaml

# 编辑配置文件，配置数据库或者端口等，如需配置请参考参考配置
# 如果不需要更改配置文件，这一步可以跳过
vim application.yaml

# 拉取Halo 镜像
docker pull halohub/halo:1.4.13

# 创建容器
docker run -it -d --name halo -p 8090:8090 -v ~/.halo:/root/.halo --restart=unless-stopped halohub/halo:1.4.13

# 容器创建完成后，进行测试
# 打开 http://ip:端口号（默认8090） 即可看到安装引导界面。
```

### 安装oneinstack

安装oneinstack是为了进行nginx管理和ssl证书部署

参考文档： https://docs.halo.run/getting-started/install/other/oneinstack/

对于这一部分官方文档写得非常完整，直接按照官方文档来。
