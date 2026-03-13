---
author: Denney Yang
pubDatetime: 2021-09-06T20:45:22+08:00
title: 使用Docker
featured: false
draft: false
tags:
- notes
description: Docker的常用命令   帮助命令  shell docker version    显示docker的版本信息 docker info       显示docker的系统信息，包括镜像和容器的数量 docker 命令 --help  帮助命令   帮助文档地址：https://docs.dock...
---

## Docker的常用命令

### 帮助命令

```shell
docker version   # 显示docker的版本信息
docker info      # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help # 帮助命令
```

帮助文档地址：https://docs.docker.com/engine/reference/commandline/docker/

### 镜像命令

**docker images 查看所有本地主机上的镜像**

```shell
docker images

# 可选项
-a, -all # 列出所有镜像
-q, -quiet # 只是显示镜像id
```
**docker search 搜索镜像**

```shell
docker search mysql

# 可选项
--filter=STARS=3000  # 搜索出来的镜像就是STARS大于3000的

docker search mysql --filter=STARS=3000
```
**docker pull 下载镜像**

```shell
docker pull mysql  # 默认下载最新版
docker pull mysql:5.7   # 下载指定版本
```

**docker rmi 删除镜像**

```shell
docker rmi -f 镜像id # 删除指定的镜像
docker rmi -f 镜像id 镜像id 镜像id  # 删除多个镜像
docker rmi -f $(docker images -aq)  # 删除全部的镜像
```

### 容器命令

**说明：我们有了镜像才可以创建容器，linux，下载一个centos镜像来测试学习**

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 可选参数说明
--name "Name"   # 容器名字（如tomcat1,tomcat2），用来区分容器 --name tomcat01
-d           	# 后台方式运行
-it          	# 使用交互方式运行，进入容器查看内容
-p           	# 指定容器端口
	 	# -p ip:主机端口:容器端口
	 	# -p 主机端口:容器端口（常用）
	 	# -p 容器端口
	 	# 容器端口
-P（大P）        # 随机指定端口

# 测试，启动并进入容器
docker run -it centos /bin/bash
ls # 查看容器内的centos，基础版本，很多命令都是不完善的
exit  # 从容器中退出主机
```

**列出所有运行的容器**

```shell
docker ps [可选参数]    # 列出当前运行的容器

# 可选参数说明
-a			# 列出当前运行的容器和历史运行过的容器
-n=?		# 显示最近？个运行的容器
-q			# 显示容器的ID
```

**退出容器**

```shell
exit		# 直接容器停止并退出
ctrl + P + Q	# 容器不停止退出 
```

**删除容器**

```shell
docker rm 容器ID					# 删除指定的容器，不能删除正在运行的容器，如果要强制删除：rm -f
docker rm -f $(docker ps -aq)	# 删除所有的容器
docker ps -a -q|xargs docker rm # 删除所有的容器
```

**启动和停止容器**

```shell
docker start 容器ID	# 启动容器
docker restart 容器ID	# 重启容器
docker stop 容器ID	# 停止当前正在运行的容器
docker kill 容器ID	# 强制停止当前容器
```

### 其它常用命令

**后台启动容器**

```shell
docker run -d 镜像名

# 当我们执行完这个命令时，使用docker ps查看时，发现没有运行的容器
# 常见的坑：docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
# nginx容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
```

**查看日志**

```shell
docker logs -tf --tail 10 容器ID # 显示容器最新的10条日志

# 可选参数说明
-tf		# 显示日志
--tail number	# 要显示日志的条数
```

**查看容器中进程信息**

```shell
docker top 容器ID
```

**查看镜像的元数据**

```shell
docker inspect 容器ID
```

**进入当前正在运行的容器**

```shell
# 我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 方式一
docker exec -it 容器ID bashshell

# 方式二
docker attach 容器ID

# 两者区别
# docker exec:进入容器开启一个新的终端（常用）
# docker attach:进入容器以前的终端，不会新创建一个终端
```

**从容器内拷贝文件到主机上**

```shell
docker cp 容器ID:容器内路径 目的的主机路径

# 测试
docker start -i 3aaad441d0d6	# 进入一个原来的容器
touch /home/test.java			# 创建一个test.java文件
exit							#退出容器或把容器切换到后台
docker cp 3aaad441d0d6:/home/test.java /home	# 将容器内的文件拷贝到主机
```

### 小结

![QQ截图20210905222238](https://i.loli.net/2021/12/03/XflBUWxkgKnYSZP.png)

```shell
attach		Attach to a running container				# 当前she1l下attach连接指定运行镜像
bui1d		Bui1d an image from a Dockerfile			# 通过Dockerfile定制镜像
commit 		Create a new image from a container changes	# 提交当前容器为新的镜像
cp 			Copy files/folders from the containers filesystem to the host path		# 从容器中拷贝指定文件或者目录到宿主机中
create		Create a new container						# 创建一个新的容器,同run，但不启动容器
diff		Inspect changes on a container's filesystem	# 查看docker容器变化
events		Get real time events from the server		# 从docker服务获取容器实时事件
exec 		Run a command in an existing container		# 在已存在的容器上运行命令
export 		Stream the contents of a container as a tar archive		# 导出容器的内容流作为一个tar归档文件[对应import]
history 	Show the history of an image				# 展示一个镜像形成历史
images		List images									# 列出系统当前镜像
import 		Create a new filesystem image from the contents of a tarbal1 	# 从tar包中的内容创建一个新的文件系统映像[对应export]	
info		Display system-wide information				# 显示系统相关信息
inspect 	Return low-level information on a container	# 查看容器详细信息
kill		Kill a running container					# kill指定docker容器
1oad		Load an image from a tar archive			# 从一个tar包中加载一个镜像[对应save]
login 		Register or Login to the docker registry server 	# 注册或者登陆一个docker源服务器
logout 		Log out from a Docker registry server		# 从当前Docker registry 退出
logs		Fetch the logs of a container				# 输出当前容器日志信息
port 		Lookup the public-facing port which is NAT-ed to PRIVATE_PORT 	# 查看映射端口对应的容器内部源端口
pause		Pause all processes within a container		# 暂停容器
ps			List containers								# 列出容器列表
pull		Pull an image or a repository from the docker registry server # 从docker镜像源服务器拉取指定镜像或者库镜像
push		Push an image or a repository to the docker registry server 	# 推送指定镜像或者库镜像至docker源服务器
restart		Restart a running container					# 重启运行的容器
rm			Remove one or more  containers				# 移除一个或者多个容器
rmi			Remove one or more images					# 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制册除]
run			Run a command in a new container			# 创建一个新的容器并运行一个命令
save		Save an image to a tar archive				# 保存一个镜像为一个tar包[对应load]
search		Search for an image on the Docker Hub		# 在docker hub中搜索镜像
start		Start a stopped containers					# 启动容器
stop		Stop a running containers					# 停止容器
tag			Tag an image into a repository				# 给源中镜像打标签
top 		Lookup the running processes of a container # 查看容器中运行的进程信息
unpause		Unpause a paused container					# 取消暂停容器
version 	show the docker version information			# 查看docker版本号
wait 		Block until a container stops，then print its exit code 		# 截取容器停止时的退出状态值
systemctl start docker									# 启动docker服务
systemctl stop docker									# 停止docker服务
systemctl restart  docker								# 重启docker服务
systemctl daemon-reload									# 守护进程重启
```

docker的命令是十分多的，上面我们学习的那些都是最常用的容器和镜像的命令，之后我们还会学习很多命令!

### 练习

> Docker部署Nginx

```shell
# 1.搜索Nginx镜像
docker search nginx			# 也可到docker hub搜索：https://hub.docker.com/

# 2.下载Nginx镜像
docker pull nginx

# 3.启动一个容器
docker run -d --name Nginx01 -p 3344:80 nginx

-d:后台运行
--name:给容器取一个名字
-p:映射主机与容器的端口（主机端口:容器端口）

# 4.测试服务是否启动
curl localhost:3344

# 5.进入容器
docker exec -it 容器ID /bin/bash

```

端口暴露的概念：

![QQ截图20210906085920](https://i.loli.net/2021/12/03/kKUipesJcZmbDo7.png)

思考问题:我们每次改动nginx配置文件，都需要进入容器内部?十分的麻烦，我要是可以在容器外部提供一个映射路径，达到在容器外部修改文件，容器内部就可以自动修改? -v 数据卷!

> Docker部署Tomcat

```shell
# 官方的使用
docker run -it --rm tomcat:9.0

# 我们之前的启动都是后台，停止了容器之后，容器还是可以查到	docker run -it --rm, 一般用来测试，用完就删除

# 下载在启动
docker pull tomcat

# 启动运行
docker run -d -p 3355:8080 --name tomcat01 tomcat

# 测试访问

# 进入容器
docker exec -it tomcat01 /bin/bash

# 发现问题：1.linux命令少了，2.没有webapps。-->阿里云镜像的原因，默认是最小的镜像，所有不必要的都剔除掉。
# 保证最小可运行的环境

# 将容器里的webapps.dist目录下的文件拷贝到webapps下
cp -r webapps.dist/* webapps

# 再次测试访问，可以访问
```

思考问题:我们以后要部署项目，如果每次都要进入容器是不是十分麻烦?我要是可以在容器外部提供一个映射路径，webapps ,我们在外部放置项目，就自动同步到内部就好了!

### 可视化

* portainer

```shell
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

* Rancher（CI/CD再用）

**什么是portainer**

Docker图形化界面管理工具，提供一个后台面板供我们操作

测试访问，登录：

![QQ截图20210906104619](https://i.loli.net/2021/12/03/diSjRQ3kBbP94VT.png)

进入容器控制面板：

![QQ截图20210906104735](https://i.loli.net/2021/12/03/cSY78IbKtdVoCMj.png)

可视化面板平时一般不使用，可以自己玩玩
