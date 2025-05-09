### 一、线程的基本使用
+ Thread
+ Runnable
+ Callable

### 二、线程生命周期
示意图：

![](/assets/images/posts/thread-lifecycle.png)

1. Java 线程一共包含6种状态
    - NEW
    - RUNNABLE（运行状态）

    		运行状态包含两种状态：READY（就绪）、RUNNING（运行中）

    - BLOCKED
    - WAITING
    - TIMED_WAITING
    - TERMINATED（终止状态）
2. Thread.start()，线程进入就绪状态，等待系统调度。系统调度后进入RUNNING 状态
3. 线程中断方式：
    - Thread.currentThread().isInterrupted(); //查看当前线程的共享变量，默认返回false
    - interrupt();
        * <font style="color:#DF2A3F;">设置线程中断方法，设置共享变量为true</font>，isInterrupted() 方法返回true
        * interrupt() 不会马上终止线程run方法，而是<font style="color:#DF2A3F;">唤醒处于阻塞状态下的线程</font>，并抛出InterruptedException异常，由异常的catch块去决定后续操作
        * 后续操作可以不处理、继续中断、或者抛出异常
        * 总结：除了设置共享变量标志位true，还会唤醒处于阻塞状态下的线程
    - Thread.interrupted();
        * 检测线程终端状态，返回当前线程的终端状态（即是否被中断）
        * 线程复位，将线程的共享变量复位成false。这是与 isInterrupted() 方法的主要区别，后者不会改变中断状态

### 三、线程安全性问题
+ <font style="color:rgb(77, 77, 77);">原子性：指一个操作是不可分割的，不可中断的，不受其他线程影响</font>
+ <font style="color:rgb(77, 77, 77);">有序性：</font>
+ <font style="color:rgb(77, 77, 77);">可见性：</font>

### 四、Synchronized 同步锁
1. 锁的范围
+ 实例锁：对象实例，锁的范围只针对当前实例有效。下面两者等价

```java
synchronized void demo(){
    
}

void demo(){
    synchronized(this){
        
    }
}
```

+ 类锁：静态方法、类对象，锁的范围针对所有对象都互斥。下面两者等价

```java
synchronized static void demo(){
    
}

void demo(){
    synchronized(SynchronizedDemo.class){
        
    }
}
```

+ 代码块



2. 锁的存储

锁的信息是存储在对象头中的，对象标记的最后3位用来存储锁的信息，如图：

![](/assets/images/posts/object-header-lock.png)

![](/assets/images/posts/object-header-lock-detail.png)



3.锁升级的过程

+ 偏向锁
    - <font style="color:rgb(77, 77, 77);">Java 6 中引入了</font>**<font style="color:rgb(77, 77, 77);">偏向锁</font>**<font style="color:rgb(77, 77, 77);">来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有</font>
    - <font style="color:rgb(77, 77, 77);"></font>
+ 轻量级锁
    -  如果偏向锁被关闭或者当前偏向锁已经已经被某个线程获取，那么这个时候如果有线程去抢占同步锁 时，锁会升级到轻量级锁  
    - <font style="color:rgb(77, 77, 77);">当有其它线程使用偏向锁对象时【没有发生锁竞争】，会将偏向锁升级为轻量级锁</font>
    - 自旋锁，jdk1.6之前是自旋10次，jdk1.6之后是自适应自旋
    - 自旋即多次CAS操作
+ 重量级锁
    - **<font style="color:rgb(77, 77, 77);">锁膨胀</font>**<font style="color:rgb(77, 77, 77);">：轻量级</font>[<font style="color:rgb(77, 77, 77);">锁升级</font>](https://so.csdn.net/so/search?q=%E9%94%81%E5%8D%87%E7%BA%A7&spm=1001.2101.3001.7020)<font style="color:rgb(77, 77, 77);">为重量级锁。</font>
    - <font style="color:rgb(77, 77, 77);">如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁</font>
    - monitor 
        * <font style="color:rgb(77, 77, 77);">Moniter称为</font>**<font style="color:rgb(77, 77, 77);">监视器</font>**<font style="color:rgb(77, 77, 77);">或者</font>**<font style="color:rgb(77, 77, 77);">管程</font>**<font style="color:rgb(77, 77, 77);">，是操作系统提供的对象。</font>
        * <font style="color:rgb(77, 77, 77);">每个Java对象都可以关联一个Moniter对象，如果使用</font>`<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">synchronized</font>`<font style="color:rgb(77, 77, 77);">给对象上锁（重量级），该对象的Mark Word中就被设置指向Moniter对象的指针</font>
        * <font style="color:rgb(77, 77, 77);">刚开始Moniter中Owner为null，当Thread-2执行synchronized(obj)后，就会将Moniter的所有者Owner置位Thread-2，Moniter只能有一个Owner</font>
        * <font style="color:rgb(77, 77, 77);">obj对象的MarkWord中最初保存的是对象的hashcode、gc年龄等信息，同时锁标志位为01，表示无锁。当获取锁后，会将这些信息保存在Moniter对象中，然后MarkWord存储的就是指向Moniter的指针，锁标志位为10（重量级锁）</font>
        * <font style="color:rgb(77, 77, 77);">在Thread-2上锁的过程中，如果Thread-1、Thread-3也来执行synchronized(obj)，就会进入EntryList，处于BLOCKED状态，Thread-2执行完同步代码块的内容后，唤醒EntryList中等待的线程来竞争锁，竞争是非公平的</font>
        * ![](/assets/images/posts/monitor.png)
    - <font style="color:rgb(77, 77, 77);">重量级锁才支持 </font>`<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">wait/notify</font>`<font style="color:rgb(77, 77, 77);">，调用后，锁直接升级为重量级锁</font>

### 五、其他
#### CAS 操作
+ ‌Java中的CAS（Compare And Swap）是一种无锁算法，用于实现多线程环境下的数据同步。‌<font style="color:rgb(51, 51, 51);">CAS操作包含三个操作数：内存位置（V）、预期原值（A）和新值（B）。如果内存位置的值与预期原值相匹配，处理器会自动将该位置的值更新为新值；否则，处理器不做任何操作。</font>
+ <font style="color:rgb(51, 51, 51);">整个比较和更新的过程是原子的，不需要使用传统的锁机制来保证线程安全，因此能够减少线程等待的时间，提高并发性能</font>

#### wait和notify
### 六、volatile 关键字
#### 作用
通过lock汇编指令保证可见性和防止指令重排序

#### 可见性问题
![](/assets/images/posts/cache-consistency.png)

    - 因为高速缓存的存在，会导致共享缓存一致性的问题，core0修改缓存后，core1读取到的缓存可能不是最新的
    - 如何保证缓存一致性的问题？
        * 缓存一致性协议，MESI，MSI
        * MESI协议：
            + <font style="color:rgb(77, 77, 77);">MESI协议是一种基于缓存失效的一致性控制协议，保证了CPU核心之间缓存一致性，也就保证了可见性</font>
            + M：	Modify 修改
            + E：	Exclusive 独占
            + S：	Shared 共享
            + I：	Invalid 失效
            + MESI协议存在的问题：
                - <font style="color:rgb(77, 77, 77);">那么，既然有了MESI，为什么还要volatile来保证缓存的可见性呢？</font>  
<font style="color:rgb(77, 77, 77);">实际上，完全遵守MESI会影响执行效率，所以现在的CPU一般在L1缓存之前还有Store Buffer缓存</font>
        * MESI协议的优化：
            + <font style="color:rgb(77, 77, 77);">为了提高执行效率，CPU也采用了异步的方式。简单来说，就是当一个核心要对一个缓存行修改之前，根据MESI，需要等待其他核心确认已经将自己缓存中的这条缓存行设置为Invalid状态才可以写入到自己缓存当中。</font>
            + **<font style="color:rgb(77, 77, 77);">Store Buffer 异步</font>**
            + <font style="color:rgb(77, 77, 77);">核心之间的消息发送和响应一定是会影响效率的，解决方式就是异步。CPU在获取到其他核心的失效确认前，会先写到Store Buffer这一结构中然后继续处理其他的任务。等到收到其他核心的Invalid Ack失效确认后，空闲时才会逐个写入到自己的缓存当中（x86处理器会以FIFO的顺序来逐个写入）。</font>
            + **<font style="color:rgb(77, 77, 77);">Invalid Queue 异步</font>**
            + <font style="color:rgb(77, 77, 77);">从上面这段可以看出，其他核心处理来自其他核心的失效请求时，也不应该立即放下手头的任务去设置自己的这条缓存行为Invalid状态。实际上会先放入Invalid Queue，然后直接回复一个Invalid Ack，等到空闲时再设置缓存行状态。（x86处理器没有Invalid Queue，当修改从Store Buffer刷入cache时，总能读到最新值）</font>
            + **<font style="#DF2A3F;">这两种异步行为就有可能会造成实际运行时的指令重排以及可见性问题</font>**
            + **<font style="color:#DF2A3F;"></font>**
+ Java内存模型（JMM, Java Memory Model）是一个非常重要的概念，它定义了程序中各个线程如何通过内存进行通信。
    - 内存屏障（硬件层面）
        * 读屏障
        * 写屏障
        * 全屏障

**内存屏障的作用**

    - 防止重排序：
        * 当一个线程对 volatile 变量进行写操作时，在写操作之前的所有普通内存操作不能被重排序到该写操作之后；
        * 当一个线程对 volatile 变量进行读操作时，在读操作之后的所有普通内存操作不能被重排序到该读操作之前。<font style="color:#DF2A3F;">这是通过在 volatile 读/写操作周围插入内存屏障来实现的</font>。
    - 保证可见性：
        * 除了防止重排序外，内存屏障还确保了缓存一致性。这意味着当一个线程修改了 volatile 变量的值，这个新值会立即刷新到主内存，并且其他线程的工作内存中对该变量的缓存将被视为无效，从而强制其他线程在下次访问该变量时必须从主内存中重新加载最新的值
    - JMM提供的内存屏障类型
        * LoadLoad 屏障
            + 形式：Load1; LoadLoad; Load2
            + 作用：确保 Load1 数据加载先于 Load2 及所有后续的数据加载操作完成。即强制 Load1 的结果必须在 Load2 开始之前被看到。
        * StoreStore 屏障
            + 形式：Store1; StoreStore; Store/XMLSchema
            + 作用：确保 Store1 数据写入先于 Store2 及所有后续的数据写入操作完成。这防止了 Store1 被重排序到 Store2 之后，有助于避免写缓冲区导致的问题。
        * LoadStore 屏障
            + 形式：Load1; LoadStore; Store2
            + 作用：确保 Load1 数据加载操作先于 Store2 及所有后续的数据写入操作完成。这意味着从内存读取的操作不能被重排序到写操作之后发生。
        * StoreLoad 屏障
            + 形式：Store1; StoreLoad; Load2
            + 作用：这是最强大的一种屏障，它不仅确保 Store1 完成后才开始 Load2，而且还会清空处理器的写缓冲区，使得所有的写操作对其他线程可见。这种屏障通常用来实现 volatile 变量的语义以及线程间的同步。

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

### 八、其他常见问题
#### 死锁
满足下面的4个条件同时满足，就会产生死锁：

+ 互斥，共享资源X和Y只能被一个线程占用
+ 占有且等待，线程T1已经取得共享资源X，在等待共享资源Y的时候，不释放共享资源X
+ 不可抢占，其他线程不能强行抢占线程T1占有的资源
+ 循环等待，线程T1等待线程T2占有的资源，线程T2等待线程T1占有的资源，就是循环等待

#### Thread.join
遵循happens-before原则。

`<font style="color:rgb(44, 44, 54);">Thread.join()</font>`<font style="color:rgb(44, 44, 54);"> 是 Java 中 </font>`<font style="color:rgb(44, 44, 54);">Thread</font>`<font style="color:rgb(44, 44, 54);"> 类的一个方法，用于等待当前线程终止。它的主要作用是使当前线程（调用 </font>`<font style="color:rgb(44, 44, 54);">join()</font>`<font style="color:rgb(44, 44, 54);"> 的线程）等待另一个线程（被调用 </font>`<font style="color:rgb(44, 44, 54);">join()</font>`<font style="color:rgb(44, 44, 54);"> 的线程）完成其执行。这在多线程编程中非常有用，可以确保某些操作在特定线程完成后才继续执行。</font>

<font style="color:rgb(44, 44, 54);">举例子：在Main线程调用t线程的join()方法，main线程或等待t线程的运行完成。</font>

```java
private static int i = 10;
public static void main(){
    Thread t = new Thread(()->{
        i = 30;
    });
    // 启动线程
    t.start();
    // t线程执行的结果对于main线程可见
    t.join();
    // 输出结果i=30
    System.out.println("i:" + i);
}
```

原理：

`<font style="color:rgb(44, 44, 54);">Thread.join()</font>`<font style="color:rgb(44, 44, 54);"> 的实现基于 </font>`<font style="color:rgb(44, 44, 54);">Object.wait()</font>`<font style="color:rgb(44, 44, 54);"> 方法。当一个线程调用另一个线程的 </font>`<font style="color:rgb(44, 44, 54);">join()</font>`<font style="color:rgb(44, 44, 54);"> 方法时，当前线程会进入等待状态，直到被调用 </font>`<font style="color:rgb(44, 44, 54);">join()</font>`<font style="color:rgb(44, 44, 54);"> 的线程终止或者超时。</font>

#### ThreadLocal
**<font style="color:#DF2A3F;">待定</font>**













