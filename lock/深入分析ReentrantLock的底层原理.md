深入分析ReentrantLock的底层原理
=============================

谈到并发，不得不谈ReentrantLock；而谈到ReentrantLock，不得不谈**AbstractQueuedSynchronized（AQS）**。
类如其名，**抽象的队列式的同步器**，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，
如常用的ReentrantLock、Semaphore、CountDownLatch等。

我们以ReentrantLock为例，深入分析ReentrantLock实现和AQS的底层原理。
