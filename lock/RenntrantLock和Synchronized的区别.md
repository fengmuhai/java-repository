RenntrantLock和Synchronized的区别.md
=========================================

本文只是说明一下两者的区别和联系，在这篇文章中不会对RenntrantLock和Synchronized的底层原理会简单说明，但不会太过深入和详细。

一、基本概念
------------------------------
#### 1.ReentrantLock 拥有Synchronized相同的并发性和内存语义。此外还多了：锁投票、定时锁等候、中断锁等候。

所以，ReentrantLock相对Synchronized多了中断的功能:

```
     当线程A和B都要获取对象O的锁定，假设A获取了对象O锁，B将等待A释放对O的锁定，

     如果使用 synchronized ，如果A不释放，B将一直等下去，不能被中断

     如果 使用ReentrantLock，如果A不释放，可以使B在等待了足够长的时间以后，中断等待，而干别的事情
```

#### 2. ReentrantLock获取锁定与三种方式：

    a)  lock(),如果获取了锁立即返回；如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁；

    b)  tryLock(), 如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；

    c)  tryLock(long timeout,TimeUnit unit)，如果获取了锁立即返回true，如果别的线程正持有锁，会等待参数给定的时间，如果获取了锁定，就返回true，如果等待超时，返回false；

    d)  lockInterruptibly:如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到锁定，或者被别的线程中断

#### 3.实现层面

synchronized是在JVM层面上实现的，可以通过一些监控工具监控synchronized的锁定，在代码执行时出现异常时，JVM会自动释放锁定；

ReentrantLock是通过代码实现的，不会自动释放锁；要保证锁定一定会被释放，就必须将unLock()放到finally{}中；

#### 4.性能

     1）在资源竞争不激烈的情况下，Synchronized的性能要优于ReetrantLock；

     2）在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；

以上的说法应该是在Synchronized以前的对比，JDK6之后对Synchronized进行了大量优化，使2者性能相近，引用![](https://cloud.tencent.com/developer/article/1458822)的一段话：

     在Synchronized优化以前，synchronized的性能是比ReenTrantLock差很多的。
     但是自从Synchronized引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了。

     在两种方法都可用的情况下，官方甚至建议使用synchronized。
     其实synchronized的优化借鉴了ReenTrantLock中的CAS技术。都是试图在用户态就把加锁问题解决，避免进入内核态的线程阻塞。


二、相同点
--------------------

1.都是通过加锁的方式同步；

2.都是可重入锁；

3.都是阻塞式的同步锁。也就是说当一个线程获取到对象锁，进入到同步块代码，那么其他线程都会阻塞在同步块之外等待，等到该线程执行完毕，释放锁才能继续获取锁执行。而进行线程阻塞和唤醒的代价是比较高的（操作系统需要在用户态与内核态之间来回切换，代价很高，不过可以通过对锁优化进行改善）。


三、不同点
---------------------
***Synchronized是Java的关键字，它从JVM层面实现了互斥锁功能；ReentrantLock是JDK1.5之后提供的API层面的互斥锁类*

#### 1.实现不同

**1）Synchronized是Java的关键字，它从JVM层面实现了互斥锁功能，自动加锁和释放锁；**

采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用，更安全

**2）ReentrantLock是JDK1.5之后提供的API层面的互斥锁类，是API层面的加锁解锁，需要手动释放锁；**

ReentrantLock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。需要lock()和unlock()方法配合try/finally语句块来完成


#### 2.锁的细粒度和灵活度

1）Synchronized锁住的是一个代码块，或者整个方法、整个类(分别是锁住代码块、实例方法、静态方法)，粒度比较大；

2）ReentrantLock可由自己的代码实现，可以跨越方法，灵活性比较大；

#### 3.等待可否中断

1）**Synchronized不可等待中断，除非抛出异常。**因为Synchronized释放锁的方式只有2种：一是出现异常自动释放锁；二是同步代码执行完，自动释放锁。

2）**ReentrantLock可以中断。**持有锁的线程长期不释放的时候，正在等待的线程可以选择**放弃等待**。

有2种方式可以放弃等待：

    a）使用lock(long timeout, TimeUnit unit)，设置超时时间，过期就自动放弃；
    b）将lockInterruptibly()方法放在代码块中，调用interrupt()方法进行中断。

#### 4.是否非公平锁

1）Synchronized是非公平锁；

2）ReentrantLock支持公平锁和非公平锁，默认是非公平锁，通过构造器传入boolean参数决定，true为公平锁，false为非公平锁。
```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }    
```

#### 5.高级功能

1）Synchronized的使用比较方便简洁，由编译器去保证锁的加锁和释放，无法通过代码获取到锁的状态信息；

2）ReentrantLock提供很多方法用来监听当前锁的信息，如：
```java
     getHoldCount() 
     getQueueLength()
     isFair()
     isHeldByCurrentThread()
     isLocked()
```

#### 6.适用情况对比

1）资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好。

2）ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。**（不知道JDK6对Synchronized优化之后，在竞争激烈情况下性能也一致了？）**


#### 总结一下ReentrantLock独有的能力：

     1.ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。

     2.ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

     3.ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。
     
     
四、Synchronized原理简述
-------------------------
详细原理分析请参看《深入分析synchronized实现原理》![]()

Synchronized在JVM中的
