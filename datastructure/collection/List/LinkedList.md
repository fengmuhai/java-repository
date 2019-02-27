LinkedList简介
=====================

1.LinkedList简述
---------------------
1）LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。

2）LinkedList 实现 List 接口，能对它进行队列操作。

3）LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。

4）LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。

5）LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。

6）LinkedList 是非同步的。


2.LinkedList构造函数
---------------------
```
LinkedList();

LinkedList(Collection<? extends E> collection);
```

3.LinkedList的API
---------------------
```
LinkedList的API
boolean       add(E object)
void          add(int location, E object)
boolean       addAll(Collection<? extends E> collection)
boolean       addAll(int location, Collection<? extends E> collection)
void          addFirst(E object)
void          addLast(E object)
void          clear()
Object        clone()
boolean       contains(Object object)
Iterator<E>   descendingIterator()
E             element()
E             get(int location)
E             getFirst()
E             getLast()
int           indexOf(Object object)
int           lastIndexOf(Object object)
ListIterator<E>     listIterator(int location)
boolean       offer(E o)
boolean       offerFirst(E e)
boolean       offerLast(E e)
E             peek()
E             peekFirst()
E             peekLast()
E             poll()
E             pollFirst()
E             pollLast()
E             pop()
void          push(E e)
E             remove()
E             remove(int location)
boolean       remove(Object object)
E             removeFirst()
boolean       removeFirstOccurrence(Object o)
E             removeLast()
boolean       removeLastOccurrence(Object o)
E             set(int location, E object)
int           size()
<T> T[]       toArray(T[] contents)
Object[]     toArray()
```

AbstractSequentialList简介
--------------------------
介绍LinkedList的源码之前，先介绍一下AbstractSequentialList。因为LinkedList是继承自AbstractSequentialList类的。

AbstractSequentialList 实现了get(int index)、set(int index, E element)、add(int index, E element) 和 remove(int index)这些方法。这些接口都是随机访问List的（LinkedList是双向链表），也相已经实现了这些接口。

此外，我们若需要通过AbstractSequentialList自己实现一个列表，只需要扩展此类，并提供 listIterator() 和 size() 方法的实现即可。
若要实现不可修改的列表，则需要实现列表迭代器的 hasNext、next、hasPrevious、previous 和 index 方法即可。


LinkedList数据结构
------------------------
**继承关系**
```
java.lang.Object
   ↳     java.util.AbstractCollection<E>
         ↳     java.util.AbstractList<E>
               ↳     java.util.AbstractSequentialList<E>
                     ↳     java.util.LinkedList<E>

public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {}
```

LinkeList与Collection的关系图
-----------------------------

![image](https://github.com/fengmuhai/JavaRepository/blob/master/datastructure/collection/images/linkedList.png)

**LinkedList的本质是双向列表**

1.LinkedList继承自AbstractSequentialList，实现了Dequeue接口；
2.LinkedList包含了两个重要成员：header和size。  
**header**是双向链表的表头，它是双向列表节点对应类的Entry的实例。Entry包含成员变量previous、next、element。其中previous是当前节点的上一个节   点，next是当前节点的下一个节点，element是当前节点的值。  
**size**是双向链表的节点个数。


LinkedList小结
----------------------------
(01) LinkedList 实际上是通过双向链表去实现的。它包含一个非常重要的内部类：Entry。Entry是双向链表节点所对应的数据结构，它包括的属性有：当前节点所包含的值，上一个节点，下一个节点。

(02) 从LinkedList的实现方式中可以发现，它不存在LinkedList容量不足的问题。

(03) LinkedList的克隆函数，即是将全部元素克隆到一个新的LinkedList对象中。

(04) LinkedList实现java.io.Serializable。当写入到输出流时，先写入“容量”，再依次写入“每一个节点保护的值”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。

(05) 由于LinkedList实现了Deque，而Deque接口定义了在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）。

(06) LinkedList可以作为FIFO(先进先出)的队列，作为FIFO的队列时，下表的方法等价：
```
队列方法       等效方法
add(e)        addLast(e)
offer(e)      offerLast(e)
remove()      removeFirst()
poll()        pollFirst()
element()     getFirst()
peek()        peekFirst()
```

(07) LinkedList可以作为LIFO(后进先出)的栈，作为LIFO的栈时，下表的方法等价：
```
栈方法        等效方法
push(e)      addFirst(e)
pop()        removeFirst()
peek()       peekFirst()
```


LinkedList的遍历
======================
**LinkedList支持多种遍历方式。建议不要采用随机访问的方式去遍历LinkedList，而采用逐个遍历的方式。**

(01) 第一种，通过迭代器遍历。即通过Iterator去遍历。
```
for(Iterator iter = list.iterator(); iter.hasNext();)
    iter.next();
```

(02) 通过快速随机访问遍历LinkedList
```
int size = list.size();
for (int i=0; i<size; i++) {
    list.get(i);        
}
```

(03) 通过另外一种for循环来遍历LinkedList
```
for (Integer integ:list) 
    ;
```

(04) 通过pollFirst()来遍历LinkedList
```
while(list.pollFirst() != null)
    ;
 ```
 
(05) 通过pollLast()来遍历LinkedList
```
while(list.pollLast() != null)
    ;
 ```
 
(06) 通过removeFirst()来遍历LinkedList
```
try {
    while(list.removeFirst() != null)
        ;
} catch (NoSuchElementException e) {
}
```

(07) 通过removeLast()来遍历LinkedList
```
try {
    while(list.removeLast() != null)
        ;
} catch (NoSuchElementException e) {
}
```

***测试结果**
```
iteratorLinkedListThruIterator：8 ms
iteratorLinkedListThruForeach：3724 ms
iteratorThroughFor2：5 ms
iteratorThroughPollFirst：8 ms
iteratorThroughPollLast：6 ms
iteratorThroughRemoveFirst：2 ms
iteratorThroughRemoveLast：2 ms
```
由此可见，遍历LinkedList时，使用removeFist()或removeLast()效率最高。但用它们遍历时，会删除原始数据；若单纯只读取，而不删除，应该使用第3种遍历方式。
**无论如何，千万不要通过随机访问去遍历LinkedList！**

参考：https://www.cnblogs.com/skywang12345/p/3308807.html
