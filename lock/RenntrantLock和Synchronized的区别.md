RenntrantLock和Synchronized的区别.md
=========================================

本文只是说明一下两者的区别和联系，在这篇文章中不会对RenntrantLock和Synchronized的底层原理会简单说明，但不会太过深入和详细。

一、基本概念
------------------------------
**1.ReentrantLock 拥有Synchronized相同的并发性和内存语义。此外还多了：锁投票、定时锁等候、中断锁等候。**

所以，ReentrantLock相对Synchronized多了中断的功能:

```
     当线程A和B都要获取对象O的锁定，假设A获取了对象O锁，B将等待A释放对O的锁定，

     如果使用 synchronized ，如果A不释放，B将一直等下去，不能被中断

     如果 使用ReentrantLock，如果A不释放，可以使B在等待了足够长的时间以后，中断等待，而干别的事情
```

**2. ReentrantLock获取锁定与三种方式：**

    a)  lock(),如果获取了锁立即返回；如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁；

    b)  tryLock(), 如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；

    c)  tryLock(long timeout,TimeUnit unit)，如果获取了锁立即返回true，如果别的线程正持有锁，会等待参数给定的时间，如果获取了锁定，就返回true，如果等待超时，返回false；

    d) lockInterruptibly:如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到锁定，或者被别的线程中断
