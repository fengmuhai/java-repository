Fail-Fast 快速失败机制
=======================

**fail-fast 机制是java集合(Collection)中的一种错误机制。**

当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。
例如：当某一个线程A通过iterator去遍历某集合如ArrayList的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出**ConcurrentModificationException异常**，产生fail-fast事件。

fail-fast解决办法
-----------------------
fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent包下的类”去取代“java.util包下的类”。
所以，本例中只需要将ArrayList替换成java.util.concurrent包下对应的类即可，如CopyOnWriteArrayList。

更多细节和深入请参照：https://www.cnblogs.com/ccgjava/p/6347425.html?utm_source=itdadao&utm_medium=referral
