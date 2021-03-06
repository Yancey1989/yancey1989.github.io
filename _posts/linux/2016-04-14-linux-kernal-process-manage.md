---
layout: post
title:  "【Linux学习笔记】之进程管理"
date:   2016-04-14 01:15:00 +0800
categories: linux
---
* content
{:toc}




## 1. 进程地址空间：虚拟地址空间
与实际物理地址无关。虚拟地址空间分为：内核空间+用户空间（TASK_SIZE）

使用页表为物理地址分配虚拟地址，内核负责将虚拟地址空间映射到物理地址空间，所以内核决定了哪些可以共享，哪些不可以共享。
使用的模型：多级分页


物理内存的分配:**伙伴系统**,目的：快速检测连续区域

进程的生命周期：运行、等待、睡眠

查看限制信息：cat /proc/self/limits

进程的两种启动方式：

* fork：当前进程的一个副本
* exec：从一个可执行的二进制文件加载另一个app

线程的启动方式：

* clone：功能强大，可以控制子进程与父进程的数据共享粒度。可以创建线程，可以使用独立的namespace

### 1.2 namespace:

* 通过fork或clone可以控制是否与父进程共享命名空间
* 通过unshare将进程某些部分从父进程分离，包括namespace

### 1.3 6种namespace:

* UTS(Unix Timesharing System,主机名),IPC（进程通信），PID（chroot进程树），NS（挂载点），NET（网络），USER

### 1.4 进程的复制：

* fork:完整副本，COW技术
* vfork:父子进程共享数据（由于fork的cow技术，vfork不被广泛采用）
* clone：产生线程，可以对父子进程之间的共享、复制进行精确控制。

### 1.5 COW技术：

* 执行fork时只负责其页表，而不是复制整个地址空间
* 涉及写操作的部分，会新申请空间，更新页表

## 2. 进程调度

### 2.1 完全公平调度器
* 虚拟时钟，完全公平调度器依赖虚拟时钟，度量等待进程在完全公平系统中所能得到的CPU时间。虚拟时钟的实现方法是根据现实的时钟以及负载情况推算出来的。
* 实现方式：基于红黑树，vruntime作为key，每次选择最左端节点来执行
