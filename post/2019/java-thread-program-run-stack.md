---
title: "Java多线程-程序运行堆栈分析"
date: 2019-08-25 12:39:19
draft: false
categories: "Java"
---
 

## class文件内容

class文件包含JAVA程序执行的字节码；数据严格按照格式紧凑排列在class文件中的二进制流，中间无任何分隔符；文件开头有一个0xcafebabe(16进制)特殊的一个标志。

![class文件内容](https://ueyao.github.io/image-hosting/blog/2019/8/program-run-heap-01.png)

## JVM运行时数据区

![jvm运行时数据区](https://ueyao.github.io/image-hosting/blog/2019/8/program-run-heap-02.png)

线程独占：每个线程都会有它独立的空间，随线程生命周期而创建和销毁
线程共享：所有线程能访问这块内存数据，随虚拟机或者GC而创建和销毁

### 方法区

JVM用来存储加载的类信息、常量、静态变量、编译后的代码等数据。
虚拟机规范中这是一个逻辑区划。具体实现根据不同虚拟机来实现。
如：oracle的HotSpot在java7中方法区放在永久代，java8放在元数据空间，并且通过GC机制对这个区域进行管理

### 堆内存

![](https://ueyao.github.io/image-hosting/blog/2019/8/program-run-heap-03.png)

堆内存还可以细分为：老年代、新生代(Eden、From Survivor、To Survivor)
JVM启动时创建，存放对象的实例。垃圾回收器主要就是管理堆内存。
如果满了，就会出现OutOfMemoryError。

### 虚拟机栈

虚拟机栈，每个线程都在这个空间有一个私有的空间。
线程栈由多个栈帧(Stack Frame)组成。
一个线程会执行一个或多个方法，一个方法对应一个栈帧。
栈帧内容包含：局部变量表、操作数栈、动态链接、方法返回地址、附件信息等。
栈内存默认最大是1M，超出则抛出StackOverflowError

### 本地方法栈

和虚拟机栈功能类似，虚拟机栈是为虚拟机执行JAVA方法而准备的，本地方法栈是为虚拟机使用Native本地方法而准备。
虚拟机规范没有规定具体的实现，由不同的虚拟机厂商去实现。
HotSpot虚拟机中虚拟机栈和本地方法栈的实现方式一样的。同样，超出大小以后也会抛出StackOverflowError。

### 程序计数器

程序计数器(Program Counter Register)记录当前线程执行字节码的位置，存储的是字节码指令地址，如果执行Native方法，则计数器值为空。
每个线程都在这个空间有一个私有的空间，占用内存空间很少。
CPU同一时间，只会执行一条线程中的指令。JVM多线程会轮流切换并分配CPU执行时间的方式。为了线程切换后，需要通过程序计数器，来恢复正确的执行位置。
