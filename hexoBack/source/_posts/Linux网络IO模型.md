---
title: Linux网络IO模型
date: 2018-09-30 15:00:16
categories: linux
tags: linux
---
Linux内核将外部所有设备当做一个文件来操作，对一个文件的读写操作会调用内核提供的系统命令，返回一个file descriptor（fd，文件描述符）。而对一个socket的操作也有对应的描述符，称为socketfd（socket描述符），描述符就是一个数字，它指向内核中的一个结构体（文件路径，数据区等一些属性）。
<!-- more -->
内容来自《Netty权威指南》

## 简介
Linux内核将外部所有设备当做一个文件来操作，对一个文件的读写操作会调用内核提供的系统命令，返回一个file descriptor（fd，文件描述符）。而对一个socket的操作也有对应的描述符，称为socketfd（socket描述符），描述符就是一个数字，它指向内核中的一个结构体（文件路径，数据区等一些属性）。
## IO模型
### 阻塞IO模型
![阻塞IO](http://ofy9dm2ii.bkt.clouddn.com/blog/io1.jpeg "阻塞IO")
最常用的IO模型就是阻塞IO模型，缺省情形下，所有文件操作都是阻塞的。我们以套接字接口为例来讲解此模型：在进程空间中调用recvfrom，其系统调用直到数据包到达且被复制到应用进程的缓冲区中或者发生错误时才返回，在此期间一直会等待，进程在从调用recvfrom开始到它反悔的整段时间内都是被阻塞的，因此被称为阻塞IO模型
### 非阻塞IO模型
![非阻塞IO模型](http://ofy9dm2ii.bkt.clouddn.com/blog/io2.jpeg "非阻塞IO模型")
recvfrom从应用层到内核的时候，如果该缓冲区没有数据的话，就直接返回一个EWOULDBLOCK错误，一般都对非阻塞IO模型进行轮询检查这个状态，看内核是不是有数据到来
### IO复用模型
![IO复用模型](http://ofy9dm2ii.bkt.clouddn.com/blog/io3.jpeg "IO复用模型")
Linux提供select/poll，进程通过将一个或多个fd传递给select或poll系统调用，阻塞在select操作上，这样select/poll可以帮助我们侦测多个fd是否处于就绪状态。select/poll是顺序扫描fd是否就绪，而且支持的fd数量有限，因此它的使用受到了一些制约。Linux还提供了一个epoll系统调用，epoll使用基于事件驱动方式代替顺序扫描，因此性能更高。当有fd就绪时，立即回调rollback。
### 信号驱动IO模型
![信号驱动IO模型](http://ofy9dm2ii.bkt.clouddn.com/blog/io4.jpeg "信号驱动IO模型")
首先开启接口信号驱动IO功能，并通过系统调用sigaction执行一个信号处理函数（此调用立即返回，非阻塞）。当数据就绪时，就为该进程生成一个SIGIO信号，通过信号回调通知应用程序调用recvfrom来读取数据并通知主循环函数处理数据。
### 异步IO
![异步IO](http://ofy9dm2ii.bkt.clouddn.com/blog/io5.jpeg "异步IO")
告知内核启动某个操作，并让内核在整个操作完成后（包括将数据从内核复制到用户自己的缓冲区）通知我们。这种模型与信号驱动模型的主要区别是：信号驱动IO有内核通知我们何时可以开始一个IO操作；yibuIO模型由内核通知我们IO操作何时已经完成
## 扩展select/poll和epoll
select/poll实现方式是轮询所有的fd（文件描述符），epoll的方式是监听fd上的callback，因此epoll相对于select有以下优点
#### FD不受限制
select单个进程能打开的FD有一定限制，默认1024，epoll不受限制（上限是操作系统的最大文件句柄数）
#### IO效率
select采用的是轮询，也就是活跃状态的和非活跃状态的都会轮询，这样一来，当FD增大后，性能线性下降。而epoll只监听活跃的FD（通过监听FD的callback实现，只有活跃的FD才会回调）所以性能不会因为FD的增加而线性下降
#### 消息传递
传统的select要通过内核把数据复制到用户空间。epoll通过mmap使内核和用户公用同一个空间
#### epoll的API更加简单
