深入分析ReentrantLock的底层原理
=============================

谈到并发，不得不谈ReentrantLock；而谈到ReentrantLock，不得不谈**AbstractQueuedSynchronized（AQS）**。
类如其名，**抽象的队列式的同步器**，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，
如常用的ReentrantLock、Semaphore、CountDownLatch等。

我们以ReentrantLock为例，深入分析ReentrantLock实现和AQS的底层原理。

一、ReentrantLock的调用过程
-----------------------------

以下是一个简单的使用ReentrantLock的例子：
```java
    public static void testReentrantLock() {
        ReentrantLock rtLock = new ReentrantLock();
        rtLock.lock();
        variable++;
        rtLock.unlock();
    }
```
可见，锁的入口和出口就是lock()和unlock()方法，我们从lock()方法开始说起。

我们先看lock的源码：
```java
    private final Sync sync;

    public void lock() {
        sync.lock();
    }
```

调用的是Sync类对象的lock方法，这是一个抽象方法：
```java
    abstract void lock();
```

抽象方法没有任何内容，让我们再看看new ReentrantLock();执行的是什么？
```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

可以看到，实际上Sync是一个抽象类，它有公平和非公平的实现类NonfairSync和FairSync，所以实际上调用的是实现类的方法：
```java
    //非公平锁同步对象
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * 执行锁定。如果获取锁成功则立即锁定，否者执行acquire方法
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

    //公平锁同步对象
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        //非公平版本的FtryAcquire。
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

```
好了，这里代码慢慢多了起来，先不做深入，下面先总结一下：

1.ReentrantLock把所有Lock接口的操作都委派到一个Sync类上，该类继承了AbstractQueuedSynchronizer：

    static abstract class Sync extends AbstractQueuedSynchronizer
    
2.Sync又有两个子类，显然是为了支持公平锁和非公平锁而定义，默认为非公平锁：    

    final static class NonfairSync extends Sync  
    final static class FairSync extends Sync
    
    
3.先列一下Reentrant.lock()方法的调用过程（默认非公平锁），便于下一步分析：  

![ReentrantLock方法调用过程](https://github.com/fengmuhai/JavaRepository/blob/master/images/lock/ReentrantLock%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E8%BF%87%E7%A8%8B.png)


二、AQS的锁实现（加锁）

AQS的锁是由一个CLH队列来实现的。

CLH队列：带头结点的双向非循环链表（FIFO）

    简单说来，AbstractQueuedSynchronizer会把所有的请求线程构成一个CLH队列，当一个线程执行完毕（lock.unlock()）时会激活自己的后继节点，但正在执行的线程并不在队列中，而那些等待执行的线程全部处于阻塞状态.
