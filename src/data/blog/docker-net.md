---
author: Denney Yang
pubDatetime: 2021-09-12T21:04:13+08:00
title: Docker网络
featured: false
draft: false
tags:
- notes
description: 理解Docker0  清空所有环境   测试  ![QQ截图20210909163221](https://s2.loli.net/2021/12/12/2f9A4lngCFvhKYb.png)  shell  docker是如何处理网络访问的   开启一个tomcat容器 docker run -...
---

## 理解Docker0

清空所有环境

### 测试

![QQ截图20210909163221](https://s2.loli.net/2021/12/12/2f9A4lngCFvhKYb.png)

```shell
# docker是如何处理网络访问的

# 开启一个tomcat容器
docker run -d -P --name tomcat01 tomcat

# 进入容器，查看ip
docker exec -it tomcat01 /bin/bash
cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	0cfed1b320cb

# 在主机上pingtomcat01
ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.329 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.046 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.049 ms

# 发现可以ping通，说明主机与容器之间可以之间通信
```

### 原理

我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡docker0

docker之间的网络是采用了桥接模式，使用了veth-pair技术！

在本机上再次测试ip addr

![QQ截图20210909171112](https://s2.loli.net/2021/12/12/URd1gonKhQE2wCF.png)

发现每启动一个容器，就会多一个veth网卡

```shell
   # 我们发现这个容器带来网卡，都是一对对的
   # veth-pair就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
   # 正因为有这个特性，veth-pair充当一个桥梁，连接各种虚拟网络设备的
   # OpenStac，Docker容器之间的连接，OVS的连接，都是使用veth-pair技术
```

容器与容器之间也是可以相互ping通的，不信的话，可以再创建一个容器，容器与容器之间相互ping

经过实验，我们可以得出docker容器之间的网络模型：

![QQ截图20210909171747](https://s2.loli.net/2021/12/12/mbXJZjAT53uPc9d.png)

结论：

1.容器之间都有一个公用的路由器：docker0

2.所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用IP

### 小结

![QQ截图20210909172258](https://s2.loli.net/2021/12/12/ftCmVNh1kBDy8GP.png)

Docker中的所有的网络接口都是虚拟的，虚拟的转发效率高

只要容器删除，对应的一对网桥就没了

## --link

问题场景：当我们的某个容器更改了ip地址时，我们需要不重启项目，也能继续使用服务。这就需要我们给容器一个固定的名字，那我们能不能直接通过容器的名字来访问服务呢？

```shell
# 当我们启动两个容器，直接通过容器的名字来进行ping是ping不通的
docker exec -it tomcat01 ping tomcat02

# 我们在启动容器的时候加上--link关键字，将两个容器之间的网络建立联系
docker run -it -P --name tomcat03 --link tomcat02 tomcat

# 我们使用tomcat03 ping tomcat02
docker exec -it tomcat03 ping tomcat02
# 我们发现可以ping通

# 我们尝试反向ping，使用tomcat02 ping tomcat03
docker exec -it tomcat02 ping tomcat03
# 我们发现不能ping通
```

--link只做了单向连接，是将建立联系的网络直接写进配置里

--link是一个老的技术，用得很少，现在一般使用自定义网络

## 自定义网络

### 查看docker所有网络

![QQ截图20210909175630](https://s2.loli.net/2021/12/12/uVC9ErPvpHYzA2W.png)

**网络模式**

bridge：桥接docker（默认，自己创建也使用bridge模式）

none：不配置网络

host：和宿主机共享网络

container：容器网络连通（用得少，局限很大)

**测试**

```shell
# 我们直接启动的命令 --net bridge，而这个就是我们的docker0
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

# docker0的特点：默认，域名不能访问，--link可以打通连接，但是--link很麻烦

# 我们可以自定义一个网络
# --driver bridge
# --subnet 192.168.0.0/16
# --gateway 192.168.0.1

[root@centos7 test]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
d59c306bb670afb8b0912cda9c6d01ad990ddf4282c13f0c0a2973f177685a41
[root@centos7 test]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
c245ed76713a   bridge    bridge    local
37d366aed4b1   host      host      local
d59c306bb670   mynet     bridge    local
b7a9f6ea64a3   none      null      local
```

![QQ截图20210909181927](https://s2.loli.net/2021/12/12/8vg1nzXCqE62ryT.png)

```shell
# 我们使用自己的网络启动两个容器
docker run -d -P --name tomcat01 mytomcat
docker run -d -P --name tomcat02 mytomcat

# 查看一下mynet网络
docker network inspect mynet
```

![QQ截图20210909184954](https://s2.loli.net/2021/12/12/f2wkqMxAlLacz7B.png)

```shell
# 我们为什么要自定义网络？
# 因为使用自定义网络的容器之间可以直接通过名字来ping通
docker exec -it tomcat01 ping tomcat02
```

我们自定义的网络docker都已经帮我们维护好了对应的关系，推荐平时这样使用网络。

### 好处

redis - 不同的集群使用不同的网络，保证集群是安全和健康的

mysql - 不同的集群使用不同的网络，保证集群是安全和健康的

## 网络连通

![QQ截图20210909191001](https://s2.loli.net/2021/12/12/ucf75ZJUMWwIT9q.png)

```shell
# 测试打通tomcat01 --mynet

# 连通之后就是将tomcat01放到了mynet网络下

# 一个容器两个ip地址!

# 类比于阿里云服务:公网ip  私网ip
```

![QQ截图20210909191320](https://s2.loli.net/2021/12/12/DJ5Ebsvm1rB2FeV.png)

```shell
# 测试连通性
docker exec -it tomcat01 ping tomcat-net-01
# tomcat01可以连接mynet网络

docker exec -it tomcat02 ping tomcat-net-01
# tomcat02依旧不能连接
```

结论：假设要跨网络操作别人，就需要使用docker network connect 连通。
