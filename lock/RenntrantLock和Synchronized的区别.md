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

以上的说法应该是在Synchronized以前的对比，JDK6之后对Synchronized进行了大量优化，使2者性能相近，引用[Java技术栈的一篇文章](https://cloud.tencent.com/developer/article/1458822)的一段话：

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



### 总结一下ReentrantLock独有的能力

     1.ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。

     2.ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

     3.ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。
     
     
     



四、Synchronized原理简述
-------------------------
详细原理分析请参看[《深入分析synchronized实现原理》]()

**JVM虚拟机支持Synchronized有2中不同方式的同步：方法级别的同步 和 方法内部一段指令序列的同步。**

通过下面这个类编译后的方法字节码说明：
```java
public class JvmSynchronized {

    int i = 0;

    public synchronized void syncMethod() {
        i++;
    }

    public void syncBlock() {
        synchronized (this) {
            i++;
        }
    }
}
```


### 1.代码块级别的同步


首先我们来看看同步代码块的字节码：
```class
public void syncBlock();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter                      // 开始进入同步代码块
         4: aload_0
         5: dup
         6: getfield      #2                  // Field i:I
         9: iconst_1
        10: iadd
        11: putfield      #2                  // Field i:I
        14: aload_1
        15: monitorexit                       // 退出同步代码块
        16: goto          24
        19: astore_2
        20: aload_1
        21: monitorexit                       // 退出同步代码块
        22: aload_2
        23: athrow
        24: return
      Exception table:
         from    to  target type
             4    16    19   any
            19    22    19   any
     //省略部分改方法的字节码
     ...
```
从字节码中可知同步语句块的实现使用的是monitorenter 和 monitorexit 指令，其中monitorenter指令指向同步代码块的开始位置，monitorexit指令则指明同步代码块的结束位置。

1.在执行monitorenter指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1；

2.相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。

#### monitorenter ：

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

     1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

     2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

     3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

#### monitorexit：

执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。




### 2.方法级别的同步

方法级的同步是隐式，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。JVM可以从方法常量池中的方法表结构(method_info Structure) 中的 ACC_SYNCHRONIZED 访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会 检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有monitor（虚拟机规范中用的是管程一词）， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor。在方法执行期间，执行线程持有了monitor，其他任何线程都无法再获得同一个monitor。如果一个同步方法执行期间抛 出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的monitor将在异常抛到同步方法之外时自动释放。

下面看看方法级同步的代码：
```class
  public synchronized void syncMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                  // Field i:I
         5: iconst_1
         6: iadd
         7: putfield      #2                  // Field i:I
        10: return
      LineNumberTable:
        line 13: 0
        line 14: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/java/jvm/JvmSynchronized;

```


可以看到，方法级别的同步并没有monitorenter和monitorexit这两条指令，取得代之的确实是ACC_SYNCHRONIZED标识，该标识指明了该方法是一个同步方法，JVM通过该ACC_SYNCHRONIZED访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

这便是synchronized锁在同步代码块和同步方法上实现的基本原理。同时我们还必须注意到的是在Java早期版本中，synchronized属于重量级锁，效率低下，因为**监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的synchronized效率低的原因**。庆幸的是在Java 6之后Java官方对从JVM层面对synchronized较大优化，所以现在的synchronized锁效率也优化得很不错了，Java 6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁，接下来我们将简单了解一下Java官方在JVM层面对synchronized锁的优化。

参考：
[https://blog.csdn.net/qq_40551367/article/details/89414446](https://blog.csdn.net/qq_40551367/article/details/89414446)
[https://blog.csdn.net/javazejian/article/details/72828483](https://blog.csdn.net/javazejian/article/details/72828483)


5.ReentrantLock原理简述
-------------------------
