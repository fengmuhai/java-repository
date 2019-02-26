ArrayList简介
======================

ArrayList是一个数组列表，底层是由数组构成，相当于动态数组。 与Java的数组相比，它能够动态增长。 
ArrayList继承自AbstractList，实现了List、RandomAccess、Cloneable、java.io.Serializable接口。

1.ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能;  
2.ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。
在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。  
3.ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆；  
4.ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输；  
5.和Vector不同，ArrayList中的操作不是线程安全的！所以，建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

![image](https://github.com/fengmuhai/JavaRepository/blob/master/datastructure/collection/images/arraylist.png)
