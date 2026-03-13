---
author: Denney Yang
pubDatetime: 2021-09-09T20:55:29+08:00
title: Docker容器数据卷
featured: False
draft: False
tags:
  - notes
description: 什么是容器数据卷  首先，回顾一下Docker的理念：将应用和环境打包成一个镜像。当我们开始使用容器时，就会产生数据，如果我们删除了容器，那容器里的数据也会随之丢失。正是因为有这样一个问题存在，所以我们需要将数据可以持久化。  容器之间可以有一个数据共享的技术，Docker容器中产生的数据，可以同步...
---
## 什么是容器数据卷

首先，回顾一下Docker的理念：将应用和环境打包成一个镜像。当我们开始使用容器时，就会产生数据，如果我们删除了容器，那容器里的数据也会随之丢失。正是因为有这样一个问题存在，所以我们需要将数据可以持久化。

容器之间可以有一个数据共享的技术，Docker容器中产生的数据，可以同步到本地，这就是卷技术。说白了就是目录挂载，将我们容器内的目录，挂载到主机上面。

总结就是，容器数据卷可以实现容器的持久化和同步操作，容器之间也可以进行数据共享！

![QQ截图20210907124749](https://s2.loli.net/2021/12/09/S9aM5y3mKIUtq7p.png)

## 使用数据卷

> 方式一：直接使用命令来挂载

```shell
docker run -it -v 主机目录:容器目录

# 测试
docker run -it -v /home/testcentos:/home centos /bin/bash

# 当容器启动后，我们可以使用docker inspect 容器ID 来查看容器挂载信息
```

![QQ截图20210907130247](https://s2.loli.net/2021/12/09/uJfYmoX87OzCnTF.png)

测试文件：

![QQ截图20210907130654](https://s2.loli.net/2021/12/09/wihIHVzF9tcvUbf.png)

再次测试：当容器停止后，在主机上修改文件是否也会同步

停止容器

在主机上修改内容

启动容器

查看文件内容是否同步

![QQ截图20210907131220](https://s2.loli.net/2021/12/09/Zoh4GqAzR8bXkQl.png)

使用容器数据卷的好处：这样我们就可以在本地主机上修改文件，而不需要进入容器。

## 实战：安装MySQL

```shell
# 获取镜像
docker pull mysql:5.7

# 运行容器，需要做数据挂载
# 安装启动mysql,需要配置密码
# 官方测试：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动容器
-d 后台运行
-p 端口映射
-v 卷挂载
--name 容器名字
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 启动成功之后，我们可以使用sqlyog来测试连接
# sqlyog连接到服务器的3310-->3310和容器内的3306映射，所以我们就可以连接上了

# 试着在本地创建一个数据库，我们再回到linux上查看，发现数据库也同样同步了
```

我们试着删除容器，看我们的数据是否还存在于本机上。

![QQ截图20210907141223](https://s2.loli.net/2021/12/09/t9SFBUxuv5WKLbe.png)

## 具名和匿名挂载

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /ect/nginx nginx

# 查看所有的volume的情况
docker volume ls
local     fba35e946c24ed9b67f31b31a1bb33505fac568418b7b9c0afec5911adfc414f

# 这里发现，有一串随机值，这一串随机值就是一个匿名的数据卷，这就是匿名挂载

# 具名挂载
docker run -d -P --name nginx02 -v jm-nginx:/etc/nginx nginx

# 查看数据卷
docker volume ls
local     ju-nginx

# 通过 -v 卷名:容器内路径，给数据卷指定一个具体的名字
# 查看一下指定数据卷的信息
docker volume inspect jm-nginx
```

![QQ截图20210907144812](https://s2.loli.net/2021/12/09/UzlItprJfDCcLKM.png)

所有的docker容器内的卷，没有指定目录的情况下都是在 **/var/lib/docker/volumes/xxxx/_data**

我们通过具名挂载可以方便的找到我们的一个卷，大多数情况都是使用**具名挂载**

```shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载？
-v 容器内路径			# 匿名挂载
-v 卷名:容器内路径			# 具名挂载
-v 宿主机路径:容器内路径		# 指定路径挂载
```

拓展：

```shell
# 通过 -v 容器内路径:ro rw 改变读写权限（容器对路径的权限）
ro   readonly # 只读，ro说明这个路径只能通过宿主机操作，容器内部无法操作
rw   readwrite  # 可读可写

# 一旦设置了容器权限，容器对我们挂载出来的内容就有限定了
docker run -d -P --name nginx02 -v jm-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v jm-nginx:/etc/nginx:rw nginx
```

## 初识DockerFile

DockerFile就是用来构建docker镜像的构建文件！命令脚本！

通过这个脚本可以生产镜像，镜像是一层一层的，就是脚本的一个一个指令，每个指令就是一层

```shell
# 创建一个dockerfile文件，名字可以随机，建议Dockerfile
# 文件中的内容 指令(大写) 参数
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "---end---"

CMD /bin/bash

# 这里的每一个命令，就是镜像的一层

# 使用dockerfile创建自定义镜像
# docker build -f dockerfile目录 -t 自定义镜像名称:版本 .(当前目录，必写)
docker build -f dockerfile01 -t mydockerimage:1.0 .
```

![QQ截图20210907153003](https://s2.loli.net/2021/12/09/I5XjpdMyWHU2ZOn.png)

```shell
# 启动我们自己的镜像
```

![QQ截图20210907153421](https://s2.loli.net/2021/12/09/NSDs8vkqMPGUpV7.png)

容器内挂载的目录，在宿主机上一定有一个对应的目录

![QQ截图20210907153704](https://s2.loli.net/2021/12/09/u4S1vp6aMt7BLCV.png)

我们可以使用docker inspect 容器ID 查看一下挂载目录的路径

![QQ截图20210907153930](https://s2.loli.net/2021/12/09/Wc6bBQGolnhMSA1.png)

这种方式我们未来使用的十分多，因为我们通常会构建自己的镜像!

假设构建镜像时候没有挂载卷，要手动镜像挂载 -v 卷名:容器内路径!

## 数据卷容器

多个mysql同步数据

![QQ截图20210907160237](https://s2.loli.net/2021/12/09/7ifNbQkqHAjz5Zd.png)

现在我们去启动三个自己创建的镜像的容器

![QQ截图20210907161135](https://s2.loli.net/2021/12/09/CfqKO4JViDdp91I.png)

启动第二个容器，使用volumes-from挂载容器一的数据

![QQ截图20210907161914](https://s2.loli.net/2021/12/09/hyJHb6nNkGUA5cp.png)

启动第三个容器，依然使用volumes-from去挂载容器一的数据

![QQ截图20210907162457](https://s2.loli.net/2021/12/09/mcrd7iXQkjWGYe8.png)

我们试着去删除容器一，看其它容器中，是否还存在同步的数据

![QQ截图20210907162824](https://s2.loli.net/2021/12/09/3abzFwy49WEhlI8.png)

通过实验发现，容器之间的数据挂载，是一个拷贝的概念，一个容器的消失，并不会影响其它同步数据的容器。只有当整个容器链上的容器都消失了，同步的数据才会随之消失

![QQ截图20210907162907](https://s2.loli.net/2021/12/09/WcqxGPZudBJLjEk.png)

## 总结

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止。

但是一旦你持久化到了本地，这个时候，本地的数据是不会删除的!
