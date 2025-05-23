---
layout: post
title: "Blocking Queue"
date: 2025-05-23 09:00:00 +0800
categories: [JUC]
tags: [java]
---

## 一、阻塞队列
BlockingQueue是Java并发包的中的一个接口，支持在多线程环境中数据的存储和传递。

### 核心特性
+ 阻塞插入和移除：如果队列满了，试图添加新元素的操作将会被阻塞（进入等待队列），直到有足够的空间（等待signal唤醒）。同样地，如果队列为空，试图移除元素的操作也会被阻塞（进入等待队列），直到队列中有元素（等待signal唤醒）。
+ 线程安全：所有操作都是线程安全的，无需额外同步机制。
+ 不允许存储null值：因为poll()方法返回null表示队列为空，所以不允许存储null值以避免混淆。
+ 可选的容量限制：有些实现允许指定最大容量，而其他则是无界的（例如LinkedBlockingQueue默认构造函数创建的是无界队列）

### BlockingQueue原理
1. BlockingQueue的内部通常通过锁（如ReentrantLock）和条件变量（Condition）来实现线程间的同步。当一个线程尝试对一个已满或已空的队列执行操作时，该线程会被挂起，直到另一个线程改变队列的状态（即增加或移除元素）。这个过程涉及到以下关键点：
    - 锁：用于确保同一时刻只有一个线程能够修改队列状态。
    - 条件变量：用于让线程在特定条件下等待（比如队列为空或满），并且可以在条件满足时重新唤醒这些线程。
2. <font style="color:rgb(44, 44, 54);">例如</font><font style="color:#DF2A3F;">ArrayBlockingQueue的实现原理</font>
    1. <font style="color:rgb(44, 44, 54);">核心成员，主要包含</font><font style="color:#DF2A3F;">一个lock和两个Condition条件控制（一个Condition一个等待队列）</font>
        1. **<font style="color:rgb(44, 44, 54);">final ReentrantLock lock;</font>**<font style="color:rgb(44, 44, 54);"> 重入锁，用来保护对队列的所有操作。</font>
        2. **<font style="color:rgb(44, 44, 54);">private final Condition notEmpty;</font>**<font style="color:rgb(44, 44, 54);"> 当队列为空时，等待获取元素的线程会在此条件上等待。</font>
        3. **<font style="color:rgb(44, 44, 54);">private final Condition notFull;</font>**<font style="color:rgb(44, 44, 54);"> 当队列已满时，等待插入元素的线程会在此条件上等待。</font>
    2. 内部逻辑细节
        1. 环形数组：为了高效利用数组空间，ArrayBlockingQueue实际上实现了环形数组的概念。即当putIndex或takeIndex到达数组末尾时，它们会被重置为0，从而形成一个循环结构
        2. put
            1. count==items.length，表示队列满了，<font style="color:rgb(44, 44, 54);">等待插入元素的线程会在此条件进入等待</font>
            2. <font style="color:rgb(44, 44, 54);">enqueue()，元素入队列，同时唤醒在notEmpty条件等待的线程</font>
        3. take
            1. count==0，表示队列空了，<font style="color:rgb(44, 44, 54);">等待获取元素的线程会在此条件上等待</font>
            2. <font style="color:rgb(44, 44, 54);">dequeue()，元素出队列，同时唤醒在notFull条件等待的线程</font>
