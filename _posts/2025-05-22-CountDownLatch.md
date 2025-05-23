---
layout: post
title: "Java CountDownLatch"
date: 2025-05-22 10:00:00 +0800
categories: [JUC]
tags: [java]
---


## CountDownLatch
CountDownLatch 是 Java 并发包J.U.C中的一个同步辅助类。它允许一个或多个线程等待，直到其他线程执行完一组操作后再继续执行

### 基本使用
```java
CountDownLatch countDownLatch = new CountDownLatch(1); //创建对象并初始化计数器
countDownLatch.await(); // 现成会被挂起
countDownLatch.countDown(); // 减少计数器
```

### 基本原理
+ CountDownLatch 的核心原理是基于 AQS（AbstractQueuedSynchronizer）实现的，<font style="color:#DF2A3F;">使用了AQS 的 state 字段作为计数器</font>
+ **初始化**：创建CountDownLatch对象，并设置初始值
+ **等待阶段**：主线程调用 await() 方法，此时若计数器不为 0，则主线程会被放入 AQS 队列中等待
+ **<font style="color:rgb(44, 44, 54);">任务执行阶段</font>**<font style="color:rgb(44, 44, 54);">：每个子线程完成自己的任务后，会调用 </font>countDown()<font style="color:rgb(44, 44, 54);"> 方法，这会使计数器减一</font>
+ **<font style="color:rgb(44, 44, 54);">解锁阶段：</font>**<font style="color:rgb(44, 44, 54);">一旦计数器变为 0，AQS 会自动唤醒所有因为调用了 await() 而处于等待状态的线程，这些线程将继续执行</font>
+ <font style="color:#DF2A3F;">区别</font><font style="color:rgb(44, 44, 54);">：</font>
    - <font style="color:rgb(44, 44, 54);">独占锁的唤醒过程：先判断头节点状态是SIGNAL，如果是则修改为0，并唤醒头结点的下一个节点</font>
    - <font style="color:rgb(44, 44, 54);">共享锁模式的节点状态是：PROPAGATE，处于这个状态下的节点，会对线程的唤醒进行传播（第一次唤醒了head节点（继续执行），head节点再唤醒第一个节点，第一个节点再唤醒第二个节点...以此类推）</font>
