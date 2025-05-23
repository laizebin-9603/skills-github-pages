---
layout: post
title: "Java Semaphore"
date: 2025-05-22 12:10:00 +0800
categories: [JUC]
tags: [java]
---


## Semaphore（信号量）
信号量（Semaphore）是操作系统中用于控制多个线程对共享资源访问的一种同步机制

### 基本概念和方法
+ **<font style="color:rgb(44, 44, 54);">许可</font>**<font style="color:rgb(44, 44, 54);">：当创建一个</font>Semaphore<font style="color:rgb(44, 44, 54);">实例时，需要指定一个许可数，这个数字代表了最多允许多少个线程同时访问受保护的资源。</font>
+ **<font style="color:rgb(44, 44, 54);">获取许可（</font>****<font style="color:#DF2A3F;">acquire()方法</font>****<font style="color:rgb(44, 44, 54);">）</font>**<font style="color:rgb(44, 44, 54);">：每当一个线程想要访问受保护的资源时，它必须首先调用</font>acquire()方法尝试获得一个许可。如果当前还有可用的许可，则该线程可以继续执行；如果没有可用的许可（即所有许可都已被其他线程占用），则调用acquire()<font style="color:rgb(44, 44, 54);">的线程将被阻塞，直到有其他线程释放了一个许可。</font>
+ **<font style="color:rgb(44, 44, 54);">释放许可（</font>****<font style="color:#DF2A3F;">release()方法</font>****<font style="color:rgb(44, 44, 54);">）</font>**<font style="color:rgb(44, 44, 54);">：一旦线程完成了对共享资源的访问，它应该调用</font>release()<font style="color:rgb(44, 44, 54);">方法来释放其持有的许可，使得其他等待的线程有机会获取许可并访问资源。</font>

### 深入理解原理
+ **<font style="color:rgb(44, 44, 54);">内部队列</font>**<font style="color:rgb(44, 44, 54);">：当没有可用的许可时，试图获取许可的线程会被放入一个等待队列中。这个队列由AQS（AbstractQueuedSynchronizer）管理，确保线程能够有序地获取到许可。</font>
+ **<font style="color:rgb(44, 44, 54);">公平性</font>**<font style="color:rgb(44, 44, 54);">：如果</font>Semaphore<font style="color:rgb(44, 44, 54);">被配置为公平模式，那么等待时间最长的线程将优先获得许可。这有助于避免饥饿问题，但可能会降低整体性能。</font>
+ **<font style="color:rgb(44, 44, 54);">非公平性</font>**<font style="color:rgb(44, 44, 54);">：</font><font style="color:#DF2A3F;">默认情况下，</font>Semaphore<font style="color:#DF2A3F;">是非公平的</font><font style="color:rgb(44, 44, 54);">，这意味着新到达的线程有机会“插队”直接尝试获取刚刚被释放的许可，而不需要排队等候。</font>

### <font style="color:rgb(44, 44, 54);">重点思路</font>
+ 唤醒过程：AQS会遍历等待队列中的节点，使用<font style="color:#DF2A3F;">LockSupport.unpark()</font>方法来恢复那些之前被挂起的线程。<font style="color:#DF2A3F;">被唤醒的线程将再次尝试获取许可</font><font style="color:rgb(44, 44, 54);">。如果此时有可用的许可（即信号量的许可计数大于0），那么该线程将成功获取许可并退出等待状态，继续执行其后续代码。</font><font style="color:#DF2A3F;">如果没有可用的许可，则该线程可能会再次进入等待状态，或者根据实现细节进行短暂的自旋后再尝试获取</font><font style="color:rgb(44, 44, 54);">。</font>
