---
layout: post
title: "Java ConcurrentHashMap"
date: 2025-05-22 15:30:00 +0800
categories: [JUC]
tags: [java]
---


## ConcurrentHashMap
### ConcurrentHashMap的实现原理
1. 整体架构
    1. JDK1.7是使用Segment和小数组HashEntry来实现，Segment本身基于ReentrantLock（重入锁）实现线程安全（即操作插入会锁住Segment）。<font style="color:#DF2A3F;">所以我们把它称做分段锁</font>
        1. <font style="color:#DF2A3F;">JDK1.7</font>的实现锁的粒度大，发生Hash冲突时性能差。
        2. 数据多需要遍历链表（性能不行）
    2. JDK1.8是由数组、单项链表、红黑数组成。默认会初始化一个长度为16的数组。
    3. ConcurrentHashMap的核心仍然是Hash表，存在hash冲突时，采用链式寻址法来解决hash冲突。当链表比较长时，数据元素的查询复杂度变成O(n)
    4. JDK1.8引入红黑树，即链表的长度大于8并且hash表的数组容量大于64时，链表会转换为红黑树（<font style="color:#DF2A3F;">如果此时数组容量小于64会优先考虑ConcurrentHashMap扩容，避免过早避免过早地将链表转换为红黑树，相比链表，它需要更多的内存空间，并且节点的创建和维护成本也更高</font>）
    5. JDK1.8使用CAS加volatile或者Synchronized的方式来保证线程安全，锁的粒度小，查询性能也更高
        1. <font style="color:#DF2A3F;">volatile+ CAS主要在初始化数组initTable时使用；</font>
        2. <font style="color:#DF2A3F;">Synchronized + CAS主要在put元素使用，主要是锁住头结点</font>
    6. 
2. put设计过程
    1. 首先会判断容器是否为空，如果容器为空，则使用volatile加CAS来初始化
    2. 如果容器不为空，根据元素计算的hash值找到相应存储位置，如果存储位置为空，则利用CAS设置该节点
    3. 如果存储位置不为空，则使用Synchronized锁住头节点，遍历链表元素新增或者替换到链表中
    4. 最后再判断是否需要转化为红黑树
3. size设计过程
    1. ConcurrentHashMap中有一个size()方法，用来获取总元素个数。但是在多线程场景中，在保证原子性的前提下，对元素个数的累加，性能非常低。ConcurrentHashMap有以下优化：
        1. 当线程竞争不激烈，直接通过CAS（baseCount + 1）来实现元素个数递增
        2. 当线程竞争激烈，使用一个数组CounterCell[]来维护元素个数。过程是：从数组随机选择一个下标，通过CAS实现原子递增，即CAS（CounterCell[x].value + 1）
    2. 因此，获取ConcurrentHashMap的元素总数，需要从两个变量累加：Sum = baseCount + CounterCell数组
    3. 此外，<font style="color:#DF2A3F;">CounterCell竞争还是激烈，也会扩容</font>。
4. ConcurrentHashMap扩容
    1. 扩容条件：
        1. 插入新元素到达容量阈值，每个ConcurrentHashMap示例都有一个负载因子，默认0.75。<font style="color:rgb(44, 44, 54);">哈希表中的元素数量超过桶数乘以负载因子时，就会触发扩容。例如，初始化容量16，当存储的键值对数量达到 12（即 16 * 0.75）时，就会触发扩容。</font>
        2. **<font style="color:#DF2A3F;">链表的长度超过了阈值（默认8）且当前数组小于64，会优先考虑扩容</font>**
    2. <font style="color:rgb(44, 44, 54);">分段扩容：</font>ConcurrentHashMap<font style="color:rgb(44, 44, 54);"> 在扩容时采用了分段迁移的方式，不是一次性地将所有旧桶中的元素迁移到新的桶中，而是每次只处理一部分。这样做可以减少锁的竞争，并允许其他读写操作同时进行</font>
    3. <font style="color:rgb(44, 44, 54);">协助扩容：</font>
        1. <font style="color:rgb(44, 44, 54);">检查当前桶是否正在被迁移</font>
        2. <font style="color:rgb(44, 44, 54);">参与迁移，线程会从当前桶开始，向后遍历直到找到下一个未迁移的桶为止，然后将这些桶中的元素重新散列并移动到新的桶位置上</font>
        3. <font style="color:rgb(44, 44, 54);">更新状态，完成迁移后，相应桶的状态会被标记为已迁移，以便后续的操作可以直接使用新的桶结构。</font>
    4. <font style="color:rgb(44, 44, 54);">完成扩容：</font>
        1. <font style="color:rgb(44, 44, 54);">当所有的桶都被迁移成功，原有的旧桶数组将被废弃，新的更大容量的桶数组成为主数组，扩容结束</font>
    5. 扩容的过程中，如何保证读/写正常？
        1. <font style="color:rgb(44, 44, 54);">如果</font>读写<font style="color:rgb(44, 44, 54);">操作到达时，该桶还没有开始迁移，则直接在旧哈希表中读/写元素</font>
        2. <font style="color:rgb(44, 44, 54);">如果</font>读写<font style="color:rgb(44, 44, 54);">操作到达时。检测到目标桶已经被标记为转发节点，则写操作会在新哈希表中进行相应的修改</font>
5. <font style="color:#DF2A3F;">为什么ConcurrentHashMap的key不允许为null？</font>
    1. 防止在并发的场景中产生歧义