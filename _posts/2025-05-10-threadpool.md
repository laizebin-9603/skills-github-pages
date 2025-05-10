---
layout: post
title: "ThreadPool"
date: 2025-05-10 10:00:00 +0800
categories: [ThreadPool]
tags: [java]
---


### 一、线程池的理解
线程池是一种用于管理和复用线程的技术，它可以帮助我们更有效地执行大量异步任务。使用线程池的好处：

+ 降低创建线程和销毁线程的性能开销
+ 线程池本身会有参数来控制线程的创建数量，可以避免资源利用率过高的问题，起到保护资源作用
+ 提高响应速度，新任务可以立即执行不需要等待线程创建

### 二、线程池类型
1. 核心概念
    1. corePoolSize：核心线程数
    2. maximumPoolSize：最大线程数，线程池最多还可以创建多少额外的线程来处理突发的任务量
    3. keepAliveTime：超时时间，对于超过核心线程数的那些线程，在空闲一段时间后会被终止以节省资源
    4. timeUnit：超时时间单位
    5. workQueue：保存执行任务的队列
    6. threadFactory：创建新线程使用的工厂
    7. handler：当线程池和任务队列都满了，新任务指定策略进行处理
2. JDK默认提供5中不同线程池的实现方式：
    1. CachedThreadPool，一种可以缓存的线程池，有以下3个特点
        1. 没有核心线程，不限制最大线程数，线程数最大可以到达：Integer.MAX_VALUE
        2. 线程存活时间是60s
        3. 阻塞队列用的是SynchronousQueue，是一种不能存储任何元素的阻塞队列，只要提交任务到这个队列，都需要分配一个工作线程来处理
    2. FixedThreadPool
        1. 核心线程数和最大线程数都是固定值，且相等
        2. 线程数等于核心线程数时，将任务加入阻塞队列等待执行
        3. 任务队列使用的是LinkedBlockingQueue，队列默认是无边界的
    3. SingleThreadExecutor
        1. 创建一个单线程化的线程池，只有唯一的工作线程来执行任务，保证所有任务都按照FIFO顺序执行
    4. ScheduledThreadPool
        1. 支持定时和周期性的任务执行
        2. 具有延时执行功能的线程池，可以用来定时调度
    5. WorkStealingPool
        1. JDK8新的线程池，内部构造一个ForkJoinPool，利用工作窃取算法并行处理请求

### 三、原理分析
1. 大概流程![](/assets/images/posts/threadpool-task.png)
2. 过程描述
    1. 略，如上图



### 四、拒绝策略
1. AbortPolicy：直接抛出异常，默认策略
2. CallerRunsPolicy：调用者所在的线程来执行任务
3. DiscardOldersPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务
4. DiscardPolicy：直接丢弃任务

### 五、线程池状态
1. 线程池状态转化![](/assets/images/posts/threadpool-status.png)
2. 状态描述
    1. ![](/assets/images/posts/threadpool-status-detail.png)

### 六、其他
1. 如何知道线程可以回收了？
    1. 因为所有工作线程都是从阻塞队列中获取执行任务的，所以，只要在在一定的时间内，阻塞队列中没有任何任务可以处理，那么这个线程就可以结束。这个过程是通过阻塞队列里面的poll()方法来实现的。
    2. poll()方法提供了超时时间和超时时间单位两个参数，当超过指定时间没有获取到任务，poll()放回null，从而终止当前线程，完成回收。
2. CPU密集型任务 和 I/O密集型任务如何配置核心数？
    1. CPU密集型任务
        1. 主要是执行计算操作，数学计算等，主要消耗的是CPU资源
        2. 核心线程数 = CPU核心数 + 1
    2. I/O密集型任务
        1. 主要涉及输入输出操作，比如文件读写、网络请求等；I/O密集型任务大部分时间处于等待状态，因此可以设置较大的最大线程数来提高并发度
        2. 核心线程数 = CPU核心数 * 2
3. 在线程池外部如何知道一个线程的任务已经执行完成？
    1. 线程池提供了isTerminated()方法，可以判断线程池的运行状态，一旦线程池的运行状态是Terminated意味着线程池的所有任务已经执行完了
    2. 线程池中有一个submit()方法，提供一个Future的返回值，通过future.get()可以获取任务返回值
    3. 可以通过CountDownLatch计数器实现，当线程执行任务完成后调用countDown()表示任务执行完成，来唤醒await()方法
    4. **<font style="color:#DF2A3F;">线程池的工作线程中有一个钩子方法afterExecute()，工作线程执行完成任务后会回调的方法</font>**



