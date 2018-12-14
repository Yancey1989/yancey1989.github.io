---
layout: post
title:  "Goroutine Design"
date:   2017-01-07 00:50:20 +0800
categories: golang
---
* content
{:toc}

# 为什么Golang需要单独开发一个Goroutine？

- 开销问题：

    POSIX的thread API虽然能够提供丰富的API，例如配置自己的CPU亲和性，申请资源等等，线程在得到了很多与进程相同的控制权的同时，开销也非常的大，在Goroutine中则不需这些额外的开销，所以一个Golang的程序中可以支持10w级别的Goroutine。
    
- 调度性能：

    在Golang的程序中，操作系统级别的线程调度，通常不会做出合适的调度决策。例如在GC时，内存必须要达到一个一致的状态。在Goroutine机制里，Golang可以控制Goroutine的调度，从而在一个合适的时间进行GC。

# Goroutine的实现原理
- 两种备选方案
    - (M:1)多个用户态的线程对应一个系统线程，它可以做快速的上下文切换。缺点是不能有效利用多核CPU
    - (1:1)一个用户态的线程对应一个系统线程，它可以利用多核机制，但上下文切换需要消耗额外的资源
- Golang的做法
    - (M:N)Golang采取了一种多对多的方案。M个用户线程对应N个系统线程，缺点增加了调度器的实现难度

- 角色：
    
    - M: 代表了系统线程，由操作系统管理
    - G：Goroutine的实体，包括了调用栈，重要的调度信息，例如channel等。
    - P：衔接M和G的调度上下文，它负责将等待执行的G与M对接。
            
        P的数量由环境变量中的`GOMAXPROCS`决定，通常来说它是和核心数对应，例如在4Core的服务器上回启动4个线程。G会有很多个，每个P会将Goroutine从一个就绪的队列中做Pop操作，为了减小锁的竞争，通常情况下每个P会负责一个队列。
- 挂起

    在Goroutine需要执行一个系统调用时，由于M是一个线程，所以必须等待它执行完才能执行其他的Goroutine。当一个新的Goroutine产生，M需要保证会有另外的一个M能够执行这个G，简单来说，当一个M进行系统调用，需要保证有另外的一个M能够继续执行Go代码。
    
    - 何时恢复？
    
        当系统调用返回时，M需要找到一个对应的P，以便能够运行Goroutine，它首先会尝试从其他线程中**窃取**一个P，如果不成功，它会将Goroutine放在一个**全局的队列**中,并将自己放在thread cache中。
        
    - 如何**窃取**
    
        这里有篇paper来描述这个设计：[work-steal](http://supertech.csail.mit.edu/papers/steal.pdf).
        
        简单来说，当队列不平衡时，会从其他队列中截取一部分Goroutine到P上进行调度。
        
        
# 参考链接
https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit#
https://morsmachine.dk/go-scheduler
(翻译)[https://www.zhihu.com/question/20862617]

