---
author: Denney Yang
pubDatetime: 2021-09-10T20:59:47+08:00
title: DockerFile
featured: False
draft: False
tags:
  - notes
description: DockerFile介绍  dockerfile是用来构建docker镜像的文件！命令参数脚本！  构建步骤：  1. 编写一个dockerfile文件 2. docker build 构建成为一个镜像 3. docker run 运行镜像 4. docker push 发布镜像（DockerHub...
---
## DockerFile介绍

dockerfile是用来构建docker镜像的文件！命令参数脚本！

构建步骤：

1. 编写一个dockerfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库！）

查看一下官方的做法：

![QQ截图20210908152259](https://s2.loli.net/2021/12/12/eMWRkPV8hu31q2y.png)

![QQ截图20210908152649](https://s2.loli.net/2021/12/12/sXhVpQ7YxoFORuH.png)

很多官方镜像都是基础包，很多功能没有，我们通常会自己搭建自己的镜像

官方既然能制作镜像，那我们也可以。

## DockerFile的构建过程

**基础知识：**

每个保留关键字（指令）都是必须是大写字母

执行从上到下顺序执行

#表示注释

每一个指令都会创建提交一个新的镜像层，并提交!

![QQ截图20210908153246](https://s2.loli.net/2021/12/12/2FX5lUpN7hdgaQq.png)

dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单

Docker镜像逐渐成为企业交付的标准，必须要掌握

DockerFile：构建文件，定义了一切的步骤，类比于源代码

DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品

Docker容器：容器就是镜像运行起来提供服务的

## DockerFile的指令

以前我们是使用别人的镜像，现在我们了解了这些指令后，我们可以试着写一个自己的镜像。

```shell
FROM					# 基础镜像，一切从这里开始构建
MAINTAINER				# 镜像是谁写的，姓名+邮箱
RUN						# 镜像构建的时候需要运行的命令
ADD						# 步骤：tomcat镜像，这个tomcat压缩包！添加内容
WORKDIR					# 镜像的工作目录
VOLUME					# 挂载的目录
EXPOSE					# 保留端口配置
CMD						# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT				# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD					# 当构建一个被继承DockerFile这个时候就会运行ONBUILD的指令，触发指令
COPY					# 类似ADD，将我们的文件拷贝到镜像中
ENV						# 构建的时候设置环境变量
```

![QQ截图20210908155355](https://s2.loli.net/2021/12/12/nHRjKgc72mS5Esr.png)

## 实战测试

DockerHub中99%的镜像都是从这个基础镜像(scratch)开始的，然后配置需要的软件和环境来进行构建

![QQ截图20210908161420](https://s2.loli.net/2021/12/12/CFK5UDOu1ehEwy8.png)

> 创建自己的centos

```shell
# 1.编写Dockerfile的文件
FROM centos
MAINTAINER Tomtree<123456@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash

# 2.通过这个文件构建镜像
# 命令 docker build -f dockerfile文件路径 -t 镜像名:[tag] .
Successfully built 9891127d4196
Successfully tagged mycentos:1.0
```

官方的centos镜像：

![QQ截图20210908163015](https://s2.loli.net/2021/12/12/sg2tvIWPKXEdZGM.png)

我们制作的镜像：

![QQ截图20210908163053](https://s2.loli.net/2021/12/12/HEO8N71S6Rgde2j.png)

我们可以使用docker history 镜像ID来查看镜像的变更历史

![QQ截图20210908163437](https://s2.loli.net/2021/12/12/dkDV8JqmzwOUQ6j.png)

我们以后就可以通过这个命令，来研究别人的镜像是怎么做的。

> CMD和ENTRYPOINT的区别

```shell
CMD						# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT				# 指定这个容器启动的时候要运行的命令，可以追加命令
```

测试CMD

```shell
# 编写dockerfile文件
vim dockerfile-cmd-test
FROM centos
CMD ["ls", "-a"]

# 构建镜像
docker build -f dockerfile-cmd-test -t cmdtest .

# run运行，发现我们的ls -a命令生效
[root@centos7 myDockerfile]# docker run cmdtest
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found

# 想追加一个命令 -l--->ls -al
docker run cmdtest -l
出现错误：
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled 


# 在使用cmd的情况下，-l替换了CMD["ls","-a"]命令，-l不是命令所以报错

# 在cmd情况下，我们要写一个完整的命令，来替换原来的命令才不会报错
[root@centos7 myDockerfile]# docker run cmdtest ls -al
total 0
drwxr-xr-x.   1 root root   6 Sep  8 09:04 .
drwxr-xr-x.   1 root root   6 Sep  8 09:04 ..
-rwxr-xr-x.   1 root root   0 Sep  8 09:04 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Sep  8 09:04 dev
drwxr-xr-x.   1 root root  66 Sep  8 09:04 etc
drwxr-xr-x.   2 root root   6 Nov  3  2020 home
lrwxrwxrwx.   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4  2020 lost+found
```

测试ENTRYPOINT

```shell
# 编写dockerfile文件
[root@centos7 myDockerfile]# vim dockerfile-entrypoint-test 
FROM centos

ENTRYPOINT ["ls","-a"]

# 构建镜像
[root@centos7 myDockerfile]# docker build -f dockerfile-entrypoint-test -t entrypoint-test .

# 运行容器
[root@centos7 myDockerfile]# docker run entrypoint-test
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt

# 我们追加命令-l，发现我们追加的命令是直接拼接在ENTRYPOINT命令后面的
[root@centos7 myDockerfile]# docker run entrypoint-test -l
total 0
drwxr-xr-x.   1 root root   6 Sep  8 09:08 .
drwxr-xr-x.   1 root root   6 Sep  8 09:08 ..
-rwxr-xr-x.   1 root root   0 Sep  8 09:08 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Sep  8 09:08 dev
drwxr-xr-x.   1 root root  66 Sep  8 09:08 etc
drwxr-xr-x.   2 root root   6 Nov  3  2020 home
lrwxrwxrwx.   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4  2020 lost+found
drwxr-xr-x.   2 root root   6 Nov  3  2020 media
```

DockerFile中很多命令都十分相似，我们最好的学习方式就是对比它们，然后测试效果。

## 实战：自建Tomcat镜像

准备镜像文件：tomcat压缩包、jdk压缩包

![QQ截图20210909103714](https://s2.loli.net/2021/12/12/Bjdm2LDpnJZRbqK.png)

编写dockerfile文件，官方命令`Dockerfile`，build会自动寻找这个文件，就不需要-f指定了

```shell
FROM centos
MAINTAINER Tomtree<123456@qq.com>

copy readme.txt /usr/local/readme.txt

ADD jdk-8u301-linux-x64.tar.gz /usr/local
ADD apache-tomcat-9.0.22.tar.gz /usr/local

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_301
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
ENV PATH $PAHT:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

ENTRYPOINT ["/usr/local/apache-tomcat-9.0.22/bin/catalina.sh","run"]
```

构建镜像

`docker build -t diytomcat .`

启动容器，挂载Tomcat目录和Tomcat日志目录

```shell
docker run -d -p 9090:8080 --name MyDIYTomcat -v /home/myDIYTomcat/test:/usr/local/apache-tomcat-9.0.22/webapps/test -v /home/myDIYTomcat/Tomcatlogs:/usr/local/apache-tomcat-9.0.22/logs diytomcat
```

写一个简单的主页，发布到test目录中

```html
<!DOCTYPE html>
<html lang="en">
      <head>
      <meta charset="UTF-8">
      <title>Title</title>
      </head>
      <body>
      <h1>My test website</h1>
      </body>
</html>
```

尝试访问网页

![QQ截图20210909130741](https://s2.loli.net/2021/12/12/RjH9yk27vtXclS4.png)

可以发现：项目成功部署，也可以成功访问

我们以后开发的步骤：掌握Dockerfile的编写，之后的一切都使用dokcer镜像来发布运行了。

## 发布自己的镜像

> Docker Hub

地址：https://hub.docker.com/

确定这个账号可以登录

在自己的服务器提交自己的镜像

```shell
[root@centos7 test]# docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
   -p, --password string   Password
      --password-stdin    Take the password from stdin
   -u, --username string   Username

# 登录自己的账号
[root@centos7 test]# docker login -u bwfish
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

登录完毕后就可以提交镜像了，就是一个命令 `docker push`

```shell
# 为了上传镜像的命名规范（作者/镜像名:tag），我们需要先使用docker tag为我们的镜像设置标签
[root@centos7 test]# docker tag 161f836c2196 bwfish/tomcat:1.0
[root@centos7 test]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
bwfish/tomcat         1.0       161f836c2196   56 minutes ago   655MB
diytomcat             latest    161f836c2196   56 minutes ago   655MB

# 上传我们的镜像
[root@centos7 test]# docker push bwfish/tomcat:1.0
The push refers to repository [docker.io/bwfish/tomcat]
48f4e66fca8d: Pushed 
0b3db1b6275f: Pushed 
ffcf8cb3634f: Pushed 
3a0cd79eb453: Pushed 
2653d992f4ef: Mounted from library/centos 
1.0: digest: sha256:2854746ba75e430ddac84bf81047c0f8c537bf81d020ee13cc3e975b06789c59 size: 1373
```

![QQ截图20210909135003](https://s2.loli.net/2021/12/12/Bb2zo4VtUOvLxNr.png)

> 阿里云镜像服务

登录阿里云

找到容器镜像服务

创建命名空间

![QQ截图20210909153536](https://s2.loli.net/2021/12/12/psucDAWVjq496C5.png)

创建容器镜像

![QQ截图20210909153615](https://s2.loli.net/2021/12/12/wgy3mBkMVIAibcz.png)

根据阿里云的文档，上传镜像到仓库

![QQ截图20210909153916](https://s2.loli.net/2021/12/12/FpWfUJjPOZ3hwDq.png)

![QQ截图20210909154556](https://s2.loli.net/2021/12/12/5gZpkjCVUflAdE8.png)

## 小结

![QQ截图20210909154916](https://s2.loli.net/2021/12/12/zqvXDlTQ6cKCs9p.png)
