---
layout: post
title: "Java Volatitle"
date: 2025-05-13 10:01:00 +0800
categories: [JUC]
tags: [java]
---



# 一、java volatile
## 作用
通过lock汇编指令保证可见性和防止指令重排序

#### 可见性问题
![](/assets/images/posts/cache-consistency.png)

- - 因为高速缓存的存在，会导致共享缓存一致性的问题，core0修改缓存后，core1读取到的缓存可能不是最新的

- - 如何保证缓存一致性的问题？

- - - 缓存一致性协议，MESI，MSI
    - MESI协议：

- - - - MESI协议是一种基于缓存失效的一致性控制协议，保证了CPU核心之间缓存一致性，也就保证了可见性
      - M：	Modify 修改
      - E：	Exclusive 独占
      - S：	Shared 共享
      - I：	Invalid 失效
      - MESI协议存在的问题：

- - - - - 那么，既然有了MESI，为什么还要volatile来保证缓存的可见性呢？
          实际上，完全遵守MESI会影响执行效率，所以现在的CPU一般在L1缓存之前还有Store Buffer缓存

- - - MESI协议的优化：

- - - - 为了提高执行效率，CPU也采用了异步的方式。简单来说，就是当一个核心要对一个缓存行修改之前，根据MESI，需要等待其他核心确认已经将自己缓存中的这条缓存行设置为Invalid状态才可以写入到自己缓存当中。
      - **Store Buffer 异步**
      - 核心之间的消息发送和响应一定是会影响效率的，解决方式就是异步。CPU在获取到其他核心的失效确认前，会先写到Store Buffer这一结构中然后继续处理其他的任务。等到收到其他核心的Invalid Ack失效确认后，空闲时才会逐个写入到自己的缓存当中（x86处理器会以FIFO的顺序来逐个写入）。
      - **Invalid Queue 异步**
      - 从上面这段可以看出，其他核心处理来自其他核心的失效请求时，也不应该立即放下手头的任务去设置自己的这条缓存行为Invalid状态。实际上会先放入Invalid Queue，然后直接回复一个Invalid Ack，等到空闲时再设置缓存行状态。（x86处理器没有Invalid Queue，当修改从Store Buffer刷入cache时，总能读到最新值）
      - **这两种异步行为就有可能会造成实际运行时的指令重排以及可见性问题**
      - 

- Java内存模型（JMM, Java Memory Model）是一个非常重要的概念，它定义了程序中各个线程如何通过内存进行通信。

- - 内存屏障（硬件层面）

- - - 读屏障
    - 写屏障
    - 全屏障

**内存屏障的作用**

- - 防止重排序：

- - - 当一个线程对 volatile 变量进行写操作时，在写操作之前的所有普通内存操作不能被重排序到该写操作之后；
    - 当一个线程对 volatile 变量进行读操作时，在读操作之后的所有普通内存操作不能被重排序到该读操作之前。这是通过在 volatile 读/写操作周围插入内存屏障来实现的。

- - 保证可见性：

- - - 除了防止重排序外，内存屏障还确保了缓存一致性。这意味着当一个线程修改了 volatile 变量的值，这个新值会立即刷新到主内存，并且其他线程的工作内存中对该变量的缓存将被视为无效，从而强制其他线程在下次访问该变量时必须从主内存中重新加载最新的值

- - JMM提供的内存屏障类型

- - - ![img](/assets/images/posts/java-volatile.png)
    - LoadLoad 屏障

- - - - 形式：Load1; LoadLoad; Load2
      - 作用：确保 Load1 数据加载先于 Load2 及所有后续的数据加载操作完成。即强制 Load1 的结果必须在 Load2 开始之前被看到。

- - - StoreStore 屏障

- - - - 形式：Store1; StoreStore; Store/XMLSchema
      - 作用：确保 Store1 数据写入先于 Store2 及所有后续的数据写入操作完成。这防止了 Store1 被重排序到 Store2 之后，有助于避免写缓冲区导致的问题。

- - - LoadStore 屏障

- - - - 形式：Load1; LoadStore; Store2
      - 作用：确保 Load1 数据加载操作先于 Store2 及所有后续的数据写入操作完成。这意味着从内存读取的操作不能被重排序到写操作之后发生。

- - - StoreLoad 屏障

- - - - 形式：Store1; StoreLoad; Load2
      - 作用：这是最强大的一种屏障，它不仅确保 Store1 完成后才开始 Load2，而且还会清空处理器的写缓冲区，使得所有的写操作对其他线程可见。这种屏障通常用来实现 volatile 变量的语义以及线程间的同步。




#### happens-before
+ happens-before是 JMM 中用来描述操作之间执行顺序的一个偏序关系，旨在提供一种足够强的内存可见性保证，定义了一些规则：
+ **<font style="color:rgb(51, 51, 51);">程序顺序规则</font>**<font style="color:rgb(51, 51, 51);">：一个线程中的每个操作，happens-before于该线程中的任意后续操作，即单线程下执行结果不变</font>
+ **<font style="color:rgb(51, 51, 51);">监视器锁规则</font>**<font style="color:rgb(51, 51, 51);">：对一个监视器锁的释放锁，happens-before于随后对这个监视器锁的加锁，即线程A释放锁后，happens-before于随后的所有线程</font>
+ **<font style="color:rgb(51, 51, 51);">volatile变量规则</font>**<font style="color:rgb(51, 51, 51);">：对一个volatile字段的写入，happens-before于任意后续对这个volatile字段的读</font>

```java
int x =10;
synchronized (this){
    x = 12;
}

// 其他线程, 读取到的x的值必定==12

x == 12
```

+ **<font style="color:rgb(51, 51, 51);">传递性</font>**<font style="color:rgb(51, 51, 51);">：如果A happens-before B，且B happens-before C，那么A happens-before C。</font>
+ **<font style="color:rgb(51, 51, 51);">线程启动规则</font>**<font style="color:rgb(51, 51, 51);">：线程的Thread.start方法调用会happens-before于该线程中的任意操作。</font>

```java
new Thread(()->{
    // 这里读取到的x的值一定是20
    x==10;
});

x=20;
t1.start();
```

+ **<font style="color:rgb(51, 51, 51);">终结器规则</font>**<font style="color:rgb(51, 51, 51);">：对象的构造函数完成happens-before于它的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(237, 238, 240);">finalize</font>`<font style="color:rgb(51, 51, 51);">方法的开始</font>
+ **<font style="color:rgb(51, 51, 51);">Thread.join() 规则</font>**<font style="color:rgb(51, 51, 51);">：线程Thread.join() happens-before 于其他线程</font>

```java
int i = 10;
Thread t = new Thread(()->{
    i = 20;
});
t.start();
t.join();
// 这里i输出的值一定是：20
System.out.print(i);


```
