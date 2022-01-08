---
layout: post
title:  "Operating System 3"
date: 2019-10-24 18:00:00
categories: OS
tags: 
  - OS 
  - Linux
cover: >- 
  http://csuzhang.info/photos/OS-8.png
---


## 四、进程通信

定义：进程之间的信息交换

**进程是资源分配的最小单位**，因此，各个进程的内存地址空间相互独立

通常来说，为了保证安全，一个进程不能直接访问另一个的地址空间

> 三种通信方式: 共享存储、 管道通信、消息传递









#### 共享存储

![OS-8](https://raw.githubusercontent.com/hanyuancheung/hanyuancheung.github.io/main/source/photos/OS-8.png)

设置一个共享空间，进程间互斥的进行访问；

分为：
1. 基于数据结构的共享存储，共享速度慢，限制多，低级通信

2. 基于存储区的共享存储，内存中划分出存储区，数据形式、存储位置都由进程来控制，速度快，高级通信

#### 管道通信pipe

> 主要用于父子进程间通信

![OS-9](https://raw.githubusercontent.com/hanyuancheung/hanyuancheung.github.io/main/source/photos/OS-9.png)

“管道”指用于连接读写进程的共享文件，称为pipe文件（内存中开辟固定大小的缓冲区）

```php
@Test
public void testPipe() throws IOException {
    // 1、获取通道
    Pipe pipe = Pipe.open();

    // 2、获取sink管道，用来传送数据
    Pipe.SinkChannel sinkChannel = pipe.sink();

    // 3、申请一定大小的缓冲区
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
    byteBuffer.put("123232142345234".getBytes());
    byteBuffer.flip();

    // 4、sink发送数据
    sinkChannel.write(byteBuffer);

    // 5、创建接收pipe数据的source管道
    Pipe.SourceChannel sourceChannel = pipe.source();
    // 6、接收数据，并保存到缓冲区中
    ByteBuffer byteBuffer2 = ByteBuffer.allocate(1024);
    byteBuffer2.flip();
    int length = sourceChannel.read(byteBuffer2);

    System.out.println(new String(byteBuffer2.array(), 0, length));

    sourceChannel.close();
    sinkChannel.close();
}
```

**注意：**

1. 半双工通信：某时间段内只能单向传输（进程互斥进行访问），若要实现双向通信，就设置两个管道

2. 数据以字符流的形式写入管道

3. 如果没有写满，就不允许读；同样，如果没读空，就不允许写

4. pipe写满时，write()系统调用被阻塞，等待读进程将数据取走后，管道变空，read()被阻塞

#### 消息传递

![OS-10](https://raw.githubusercontent.com/hanyuancheung/hanyuancheung.github.io/main/source/photos/OS-10.png)

数据交换以格式化消息Message为单位，通过OS的“发送/接收消息”的原语进行

**消息 = 消息头 + 消息体**
（消息头：发送ID、接收ID、消息类型、长度）

## 五、多线程概念模型

引入线程的目的：提高并发度

传统的进程是程序执行流的最小单位；有了多线程，线程就成了程序执行流的最小单位

**进程是资源分配的最小单位，线程是调度的最小单位**

1. 进程间并发，需要切换进程的运行环境，开销大

2. 线程间并发，若在同一进程内，不切换环境，开销小

#### 属性要点

1. 每个线程都由独立的ID、线程控制块TCB

2. 线程几乎不拥有系统资源

3. 同一进程的线程之间共享进程的资源

#### 实现方式

用户级线程、内核级线程

> 用户级线程——多对一模型
由应用程序通过线程池来管理实现。**用户级线程中，线程切换可在用户态下完成**

理解：**从用户角度可以看到的线程**

> 内核级线程——一对一模型
由操作系统完成该进程的管理。**内核级线程中，线程切换需要在核心态下完成**

理解：**从OS内核视角看到的线程**

> 二者组合
将n 个用户级线程映射到m个内核级线程（n >= m）

克服了多对一模型并发度不高的缺点，克服了一对一模型中每个用户进程占用太多资源的缺点

通过下图进行小节下：

![OS-11](https://raw.githubusercontent.com/hanyuancheung/hanyuancheung.github.io/main/source/photos/OS-11.png)

## 六、CPU的调度

从就绪队列中按照一定的算法选择一个进程并将CPU时间分配给他调度，实现进程的并发

**三种调度:高级调度(作业调度)、中级调度(内存调度)、低级调度(进程调度)**

> 高级调度
是外存与内存之间的调度

按一定的原则从外存上处于后备队列的作业中挑选一个或多个作业，分配内存等必要资源，建立进程

> 中级调度
提高系统内存利用率、系统吞吐量

决定将哪个处于挂起状态的进程重新调入内存，一个进程可能多次被调出、调入内存，**发生频率高于高级调度**

> 低级调度
最基本的一种调度

按照某种策略或方法，从就绪队列中选取一个进程分配CPU时间，**调用的频率很高**

![OS-12](https://raw.githubusercontent.com/hanyuancheung/hanyuancheung.github.io/main/source/photos/OS-12.png)

