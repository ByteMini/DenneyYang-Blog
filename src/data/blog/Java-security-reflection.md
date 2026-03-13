---
author: Denney Yang
pubDatetime: 2022-07-20T10:25:13+08:00
title: Java安全之反射
featured: false
draft: false
tags:
- notes
description: 反射的理解  在Java中我们通常是通过类示例化一个对象，然后通过对象使用类中的方法。反射就是将顺序反过来，对象通过反射获取相应的类，类通过反射获取类中的所有方法，从而可以使用这个类中的其他方法。这样通过反射我们给Java这种静态语言赋予了动态特征，P神对动态特征的定义“一段代码，改变其中的变量，将...
---

## 反射的理解

在Java中我们通常是通过类示例化一个对象，然后通过对象使用类中的方法。反射就是将顺序反过来，对象通过反射获取相应的类，类通过反射获取类中的所有方法，从而可以使用这个类中的其他方法。这样通过反射我们给Java这种静态语言赋予了动态特征，P神对动态特征的定义“一段代码，改变其中的变量，将会导致这段代码产生功能性的变化，我称之为动态特征”。

![Java001](https://s2.loli.net/2022/07/20/erYUiE5P9jhlnov.png)

## 反射里几个重要的方法

* 获取类的方法：forName
* 实例化类对象的方法：newInstance
* 获取函数的方法：getMethod
* 执行函数的方法：invoke

## 获得类的三种方法

获得类其实就是获得java.lang.Class对象。

1.通过上下文中的某个类的实例化对象获得类：

```java
System.out.printf("Empty block initial %s\n", this.getClass());
```

2.通过已经加载的类的class属性获得，这个其实不算是反射

```java
System.out.printf("Static initial %s\n", TrainPrint.class);
```

3.当我们知道某个类的名字时，可以通过forName来获取

```java
Class clazz = Class.forName("java.lang.Runtime");
```

## 类初始化的顺序

这个问题由forName有两个重载函数引出。

```java
# forName的两个重载函数
public static Class<?> forName(String className)
public static Class<?> forName(String name, boolean initialize, ClassLoader loader)
第一个重载函数算是第二个重载函数的封装
在第二个重载函数中，name:要获得的类的类名，initialize:是否初始化，loader:jvm加载类的方式
```

上面的initialize变量表示的初始化为类的初始化，那么类什么时候发生初始化呢？

现有如下代码：

```java
public class TrainPrint {
    {
        System.out.printf("Empty block initial %s\n", this.getClass());
    }

    static {
        System.out.printf("Static initial %s\n", TrainPrint.class);
    }

    public TrainPrint(){
        System.out.printf("Initial %s\n", this.getClass());
    }

    public static void main(String[] args) {
        TrainPrint trainPrint = new TrainPrint();
    }
}
```

运行结果：

![Java002](https://s2.loli.net/2022/07/20/jDZqiFuUo2YzQfK.png)

从运行结果来看，先执行了static区域的代码，然后执行{}区域的代码，最后执行了构造函数里的代码。

static里的代码就是在类初始化时执行的，{}区域的代码是在super()函数后，构造函数前执行的。

![Java003](https://s2.loli.net/2022/07/20/2kSUL8JMeRZIyYs.png)

## 使用反射获得的类

我们使用反射的最大目的就是为了获得我们想要的方法，当我们通过反射获得类时，我们可以继续通过反射来获得类中的属性、方法，也可以实例化这个类，并调用方法。我们可以使用class.newInstance()来实例化一个类，class.newInstance()的作用就是调用这个类的无参构造函数，但当我们使用newInstance方法来实例化一个类时可能不会成功，不成功的原因可能有：1.使用的类没有无参构造函数 2.使用的类的构造函数是私有的。

通过反射我们获得的最常见的，用来构造命令执行Payload的类就是java.lang.Runtime，但我们不能通过如下方式来执行命令：

```java
Class clazz = Class.forName("java.lang.Runtime");
clazz.getMethod("exec", String.class).invoke(clazz.newInstance(), "id");
```

这样做会报错，因为我们翻看Runtime类的源码，发现Runtime的构造函数时私有的。

```java
# Runtime类部分源码
public class Runtime {
	private static Runtime currentRuntime = new Runtime();

	public static Runtime getRuntime() {
        	return currentRuntime;
    	}
	
	private Runtime() {}
}
```

获得Runtime类实例，是通过一个getRuntime静态方法来获得的，这是设计模式中的单例模式。所在我们可以通过getRuntime静态方法来获得Runtime实例，从而执行exec方法。

```java
Class clazz = Class.forName("java.lang.Runtime");
Method execMethod = clazz.getMethod("exec", String.class);
Method getRuntimeMethod = clazz.getMethod("getRuntime");
Object runtime = getRuntimeMethod.invoke(clazz);
execMethod.invoke(runtime, "calc.exe");
```

对于上面的clazz.getMethod("exec", String.class)函数就是用来获取Runtime类中的exec方法的，因为Java支持重载，我们不能仅通过函数名来获得函数，还有明确函数的参数类型，所以第一个参数为要获得的函数名称，第二参数为函数的参数类型列表，这里使用exec函数重载中最简单的一个：public Process exec(String command)。

对于上面的invoke()方法，就是执行相应的方法，invoke()中的第一个参数：如果这个方法是一个普通方法，那么第一个参数就是类对象；如果这个方法是一个静态方法，那么第一个参数就是类。如getRuntime是一个静态方法，所以传入类：getRuntimeMethod.invoke(clazz)，exec为普通方法所以传入类对象：execMethod.invoke(runtime, "calc.exe")。invoke()函数的使用可以这样理解：正常的执行顺序是：object.method(arg1, arg2, ...)，反射就是：method.invoke(object, arg1, arg2, ...)。顺序反过来了。

## 其他实例化类的方法

### getConstructor

如果一个类中没有无参构造方法，也没有类似单例模式里的静态方法，我们应该怎样通过反射实例化该类呢？此时我们可以用一个新的反射方法getConstructor。这个方法就是用来获得构造函数的，我们只需要传入参数类型列表就可以确定是哪一个构造函数。

除了Runtime可以用来执行命令外，还可以使用ProcessBuilder来执行命令，ProccessBuilder中就只要两种有参构造函数：

```java
public ProcessBuilder(List<String> command) {
        if (command == null)
            throw new NullPointerException();
        this.command = command;
}

public ProcessBuilder(String... command) {
	this.command = new ArrayList<>(command.length);
        for (String arg : command)
            this.command.add(arg);
}
```

对于ProccessBuilder的第一个构造函数，我们可以这样来进行命令执行：

```java
public static void test1(){
        try{
            Class clazz = Class.forName("java.lang.ProcessBuilder");
            Constructor clazzCons = clazz.getConstructor(List.class);
            Method start = clazz.getMethod("start");
            Object clazzIns = clazzCons.newInstance(Arrays.asList("calc.exe"));
            start.invoke(clazzIns);
        }catch (Exception e){

        }
}

# start()用来执行方法
```

对于ProccessBuilder的第二个构造函数，我们可以这样来进行命令执行：

```java
public static void test2(){
        try {
            Class clazz = Class.forName("java.lang.ProcessBuilder");
            Constructor clazzCons = clazz.getConstructor(String[].class);
            Method start = clazz.getMethod("start");
            Object clazzIns = clazzCons.newInstance(new String[][]{{"calc.exe"}});
            start.invoke(clazzIns);
            System.out.println("test2");
        }catch (Exception e){

        }
}

# 这里newInstance()为什么是二位数组？因为newInstance()本身接受的参数是可变的，同时getConstructor接受也是可变的参数，所以叠加起来就是二维数组
```

### getDeclaredConstructor

对于一个方法或构造方法是私有的，如果我们非要执行它，该怎么办呢？

对于getDeclared系列的反射与普通getMethod、getConstructor的区别：

* getMethod系列方法可以获取当前类以及当前类从其父类继承来的所有公共方法。
* getDeclaredMethod系列方法可以获取当前类中的所有方法，包括私有方法，但不能获取当前类从其父类继承的方法。
* getConstructor方法和getMethod方法类似，可以获取当前类的公共构造方法。
* getDeclaredConstructor方法和getDeclaredMethod方法类似，可以获取当前类的所有构造方法，包括私有构造方法

根据以上的结论，我们就可以使用getDeclaredConstructor()方法来获得私有的构造方法，如对于前面的Runtime我们可以这样构造命令执行：

```java
public static void test3() {
        try {
            Class clazz = Class.forName("java.lang.Runtime");
            Constructor clazzCons = clazz.getDeclaredConstructor();
            clazzCons.setAccessible(true);
            Object clazzIns = clazzCons.newInstance();
            Method exec = clazz.getMethod("exec", String.class);
            exec.invoke(clazzIns, "calc.exe");
            System.out.println("test3");
        } catch (Exception e) {
            System.out.println(e);
        }
}

# 当我们获得一个私有方法后，我们还必须使用setAccessible(true)来修改作用域，否则我们任然无法调用。
```

## 总结

这是学习P神的Java安全漫谈的一个小结，反射是学习Java安全的基础，反射的最大一个目的就是通过现有的对象获得我们想要的方法，从而调用这个方法来达到我们的目的。

## 参考文档

phith0n的Java安全漫谈
