---
title: IO
katex: false
tags: java
categories: java
abbrlink: 25718
date: 2021-07-31 10:45:35
---
I/O(Input/Output):输入/输出

根据冯.诺伊曼结构，计算机结构分为5大部分：运算器、控制器、存储器、输入设备、输出设备

输入设备(input device) --> (cpu(控制器、运算器) < -- > 存储器) --> 输出设备(output device)
![计算机结构](http://whh.plus:7007/images/2021/07/31/20190624122126398.jpg)

输入设备(比如键盘) 和 输出设备(比如鼠标)都属于外部设备。网卡、硬盘既属于输入设备，也属于输出设备

输入设备向计算机输入数额据，输出设备接收计算机输出的数据。

**I/O描述了计算机系统与外部设备之间通信的过程**

为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为用户空间(User space)和内核空间(Kernel space)

允许的应用程序都是运行在用户空间，只有内核空间才能进行系统级别的资源有关的操作，比如如文件管理、进程通信、内存管理等。要进行IO操作，一定要依赖内核空间的能力，**用户空间的程序不能直接访问内核空间**，想要执行IO操作时，由于没有执行这些操作的权限，只能发起系统调用请求操作系统帮忙完成，用户进程想要执行IO操作的话，必须通过**系统调用**来间接访问内核空间

IO操作分为两个：IO调用和IO执行。IO调用是由进程发起的，IO执行是操作系统的工作。

磁盘IO(读写文件) 和 网络IO(网络请求和相应)

从应用程序的角度说，应用程序对操作系统的内核发起IO调用(系统调用)，操作系统负责的内核执行具体的IO操作，实际上应用程序只是发起了IO操作的调用而已，具体的IO执行是由操作系统的内核来完成的

当应用程序发起I/O调用后，会经历两个步骤：
1. 内核等待I/O设备准备好的数据
2. 内核将数据从内核空间拷贝到用户空间

在有数据从内核缓冲区拷贝到进程缓冲区前，进程缓冲区处于不可读状态。IO未就绪

## 常见的IO模型
UNIX系统下，IO模型一共有5种：同步堵塞I/O、同步非堵塞I/O，I/O多路复用、信号驱动I/O和异步I/O

## JAVA种3种常见IO模型

### BIO(Blocking I/O)
**BIO 属于同步堵塞I/O模型**

同步堵塞IO模型中，应用程序发起read调用后，会一直堵塞，直到在内核把数据拷贝到用户空间。
![BIO](http://whh.plus:7007/images/2021/07/31/6a9e704af49b4380bb686f0c96d33b81tplv-k3u1fbpfcp-watermark.png)
在客户端连接数量不高的情况下，是没有问题的。但是，当面对十万甚至百万连接的时候，传统BIO模型不能为力。因此需要一种更高效的IO处理模型来应对更高并发量

### NIO(Non-blocking I/O)
JAVA中NIO于Java1.4引入。对应`java.nio`包，提供了`Channel、Selector、Buffer`等抽象。支持面向缓冲，基于通道的I/O操作方法。对于高负载、高并发的网络应用，应使用NIO。

Java中NIO可以看作是I/O多路复用模型，也有人认为，NIO属于同步非阻塞IO模型

### **同步非阻塞IO模型**
同步非阻塞IO模型中，应用程序会一直发起read调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是堵塞的。但相比同步阻塞IO模型，同步非阻塞IO模型通过轮询操作，避免了一直阻塞。但是应用程序不断继续宁IO系统调用轮询数据是否已经准备好了的过程也是十分耗费CPU资源的。
![NI0](http://whh.plus:7007/images/2021/07/31/bb174e22dbe04bb79fe3fc126aed0c61tplv-k3u1fbpfcp-watermark.png)

### **I/O多路复用模型**
IO多路复用模型中，线程首先发生select调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起read调用。read调用的过程(数据从内核空间->用户空间)还是堵塞的
![I/O多路复用](http://whh.plus:7007/images/2021/07/31/88ff862764024c3b8567367df11df6abtplv-k3u1fbpfcp-watermark.png)

> 目前支持IO多路复用的系统调用，有select，epoll等等。select系统调用，是目前几乎在所有的操作系统上都有支持 
- select调用：内核提供的系统调用，支持一次查询多个系统调用的可用状态。几乎所有操作系统都支持
- epoll调用：Linux2.6内核，属于select调用的增强版本，优化了IO的执行效率

IO多路复用模型，通过减少无效的系统调用，减少了对CPU资源的消耗。

Java中的NIO，有一个非常重要的选择器(selector)的概念，也可称为多路复用器。通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务
![](http://whh.plus:7007/images/2021/07/31/0f483f2437ce4ecdb180134270a00144tplv-k3u1fbpfcp-watermark.png)

## AIO(Asynchronous I/O)异步IO模型
AIO 也就是NIO 2. Java7引入了的NIO的改进版，异步IO模型

异步IO是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成之后，操作系统会通知相应的线程进行后续的操作。
![AIO](http://whh.plus:7007/images/2021/07/31/3077e72a1af049559e81d18205b56fd7tplv-k3u1fbpfcp-watermark.png)

AIO的应用不是很广泛，Netty之前尝试过用AIO，后来放弃了，Netty使用AIO，性能却没有提升多少

![小结](http://whh.plus:7007/images/2021/07/31/33b193457c928ae02217480f994814b6.png)

参考
[JavaGuide-IO模型](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/IO%E6%A8%A1%E5%9E%8B)
[程序员应该这样理解IO](https://www.jianshu.com/p/fa7bdc4f3de7)
[10分钟看懂， Java NIO 底层原理](https://www.cnblogs.com/crazymakercircle/p/10225159.html)
[IO模型知多少](https://www.cnblogs.com/sheng-jie/p/how-much-you-know-about-io-models.html)