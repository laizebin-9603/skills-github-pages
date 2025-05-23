---
layout: post
title: "Java CyclicBarrier"
date: 2025-05-22 13:10:00 +0800
categories: [JUC]
tags: [java]
---

## CyclicBarrier
CyclicBarrier是 Java 并发包 java.util.concurrent 中的一个同步辅助类，它允许一组线程相互等待，直到所有线程都到达一个共同的屏障点（barrier）。CountDownLatch 不同的是，CyclicBarrier 在触发后可以重用，这意味着它可以用来实现循环或者重复的同步操作。

### 基本概念
+ CyclicBarrier(int parties)：创建一个新的 CyclicBarrier，设置需要等待的线程数目为 parties
+ CyclicBarrier(int parties, Runnable barrierAction)：除了设定需要等待的线程数目外，还可以提供一个 Runnable，当所有线程都到达屏障点时自动执行这个 Runnable。

### 原理
+ 阻塞过程
    1. 初始化：当你创建一个 CyclicBarrier 实例时，你需要指定参与同步的线程数量（称为 parties）。这个数字表示需要多少个线程调用await() 方法后才能解除当前的屏障。
    2. 调用 await() 方法：当一个线程到达屏障点并调用await() 方法时，该方法会首先将内部计数器减一，并检查是否所有参与的线程都已经到达屏障点。如果还没有，则当前线程会被阻塞，放入一个等待队列中。
    3. 进入等待状态：具体来说，当线程调用await() 方法时，CyclicBarrier 会通过底层的同步机制（通常是基于 AQS）来挂起这个线程。这通常涉及到使用LockSupport.park() 方法来暂停线程的执行，使它进入等待状态，直到被显式唤醒或由于其他原因（如中断）而恢复执行。
+ 唤醒过程
    1. 最后一个线程到达屏障点：一旦最后一个线程调用了await() 方法，使得内部计数器达到零，CyclicBarrier 就知道现在所有线程都已经到达了屏障点。
    2. 触发屏障动作（如果有的话）：如果在创建CyclicBarrier 时提供了一个Runnable 对象作为屏障动作，那么此时将会执行这个Runnable。这个动作通常用于执行一些在所有线程到达屏障点之后需要进行的操作，比如合并结果或者更新共享状态等。
    3. 重置内部状态：在执行完屏障动作之后，CyclicBarrier 会重置其内部计数器为初始值（即你设定的parties），以便它可以被再次使用。这是因为CyclicBarrier支持重用，也就是说，它可以在一轮同步完成后继续用于下一轮同步。
    4. 唤醒所有等待的线程：CyclicBarrier 会遍历队列中的所有节点，并使用LockSupport.unpark() 方法逐个唤醒这些线程。这意味着之前被阻塞的所有线程现在都将从它们调用await() 方法的地方恢复执行，继续执行它们后续的代码。


