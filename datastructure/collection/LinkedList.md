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
