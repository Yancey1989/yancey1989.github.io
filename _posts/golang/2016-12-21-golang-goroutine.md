---
layout: post
title:  "Golang中的并发编程"
date:   2016-12-21 00:00:20 +0800
categories: golang
---
* content
{:toc}

# Golang中的并发编程
Golang语言中，并发使用的协程的机制，实现起来也是十分的方便，使用`go`关键字即可。

```
func main() {
    ...
    go fun(){
        fmt.Println("Hi, Here is a goroutine.")
    }()
    ...
}
```

# channel
Channel是Golang语言内置的一种管道类型，可以读取和发送数据。
goroutine之间的同步控制主要是通过channel来控制的

## channel的基本语法

1. 定义一个channel

```
c := make(chan int) 
cWithBuffer := make(chan int, 10) //带缓冲的channel
```

2. 向channel发送消息

```
c <- 1
```

3. 从channel读取消息

```
    <- c
    v, ok := <- c
```

## 如何控制goutine的并行数量
当我们需要去控制goroutine的并行数量时，需要使用到带Buffer的Channel了:

```
channelWithBuffer := make(chan int, 1)
```
这表示Channel中最大可以有1个元素

```
c := make(chan int, 1)
c <- 1 // does not block.
c <- 1 // block... until another goroutine recieves from the channel.
```

## close()的作用
当我们要关闭一个channel时候，可以使用close()函数

```
close(c)
```

close函数必须由Sender一方进行调用，当关闭一个channel后，Receiver可以继续将未读完的element从channel中读出来，当最后一个element被读出后，再进行读取操作会导致panic

```
var c = make(chan int, 10)

func main() {
    c <- 1
    close(c)
    v, ok := <-c
    fmt.Println(v, ok)
    v, ok = <-c
    fmt.Println(v, ok)
}
1 true
0 false
```

## 什么是单向chan
类似C++中的Const关键词，有时需要显示的声明某个函数不允许向channel中写入或者读取数据，这时会用到了单向channel

```
func recive(c1 <-chan int, c2 chan<- int) {
    v := <-c1
    c2 <- v
}
func send(c chan<- int) {
    c <- 1
}
func main() {
    c1 := make(chan int, 1)
    c2 := make(chan int, 1)
    send(c1)
    recive(c1, c2)
    fmt.Println(<-c2)
}
```

## timeout机制
使用`select`可以实现goutinue的超时控制

```
func main() {
    c := make(chan int, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c <- 1
    }()
    select {
    case v := <-c:
        fmt.Println(v)
    case <-time.After(time.Second):
        fmt.Println("time out")
    }
}
```
