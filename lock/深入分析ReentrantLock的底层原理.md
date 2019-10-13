深入分析ReentrantLock的底层原理
=============================

谈到并发，不得不谈ReentrantLock；而谈到ReentrantLock，不得不谈**AbstractQueuedSynchronized（AQS）**。
类如其名，**抽象的队列式的同步器**，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，
如常用的ReentrantLock、Semaphore、CountDownLatch等。

我们以ReentrantLock为例，深入分析ReentrantLock实现和AQS的底层原理。

一、ReentrantLock的调用过程
-----------------------------

1.ReentrantLock把所有Lock接口的操作都委派到一个Sync类上，该类继承了AbstractQueuedSynchronizer：

    static abstract class Sync extends AbstractQueuedSynchronizer
    
2.Sync又有两个子类，显然是为了支持公平锁和非公平锁而定义，默认为非公平锁：    

    final static class NonfairSync extends Sync  
    final static class FairSync extends Sync
    
    
3.先理一下Reentrant.lock()方法的调用过程（默认非公平锁）：  

![ReentrantLock方法调用过程](https://github.com/fengmuhai/JavaRepository/blob/master/images/lock/ReentrantLock%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E8%BF%87%E7%A8%8B.png)


二、AQS的锁实现（加锁）

AQS的锁是由一个CLH队列来实现的。

CLH队列：带头结点的双向非循环链表（FIFO）

    简单说来，AbstractQueuedSynchronizer会把所有的请求线程构成一个CLH队列，当一个线程执行完毕（lock.unlock()）时会激活自己的后继节点，但正在执行的线程并不在队列中，而那些等待执行的线程全部处于阻塞状态.
