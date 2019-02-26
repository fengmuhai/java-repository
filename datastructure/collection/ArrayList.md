ArrayList简介
======================

1.简述
----------------------

**ArrayList是一个数组列表，底层是由数组构成，相当于动态数组。 与Java的数组相比，它能够动态增长。 
ArrayList继承自AbstractList，实现了List、RandomAccess、Cloneable、java.io.Serializable接口。**

1）ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能；

2）ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess为List提供快速随机访问功能，即通过下标获取元素； 

3）ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆；

4）ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输；

5）和Vector不同，ArrayList中的操作不是线程安全的！所以，建议在单线程中使用ArrayList，多线程中可以选择Vector或者CopyOnWriteArrayList。

![image](https://github.com/fengmuhai/JavaRepository/blob/master/datastructure/collection/images/arraylist.png)


2.构造方法
----------------------
`
//默认构造方法
ArrayList();

//指定ArrayList容量大小，当添加数据导致容量不足时，扩容为原来的1.5倍，ArrayList使用的是一种延迟分配空间的动态扩容机制。
//比如：初始容量为10，要插入12个，那么在插入第11个元素时会将数组容量变为：10*1.5=15，之后可继续插入；如果要插入20个元素，那么插入11元素时扩容为15，插入第16个元素时，才扩容为15*15=22。
ArrayList(int capacity);

//创建一个包含collection的ArrayList
ArrayList(Collection<? extends E> collection);

`
