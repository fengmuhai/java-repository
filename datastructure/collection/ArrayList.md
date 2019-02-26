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


2.数据结构
----------------------
```
java.lang.Object
   ↳     java.util.AbstractCollection<E>
         ↳     java.util.AbstractList<E>
               ↳     java.util.ArrayList<E>

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```
**ArrayList包含了2个重要的属性：elementData、size**

1）elementData是Object[]数组类型，用来保存添加的列表元素。elementData是一个动态数组，可以通过构造函数ArrayList(capacity)指定初始大小，不指定则默认初始容量为10。elementData会根据添加的元素增加动态增长，增长方式见第3点；

2）size是数组实际的大小。

![image](https://github.com/fengmuhai/JavaRepository/blob/master/datastructure/collection/images/arraylist.png)


3.构造方法
----------------------
```
//默认构造方法
ArrayList();

//指定ArrayList容量大小，当添加数据导致容量不足时，扩容为原来的1.5倍，ArrayList使用的是一种延迟分配空间的动态扩容机制。
//比如：初始容量为10，要插入12个，那么在插入第11个元素时会将数组容量变为：10*1.5=15，之后可继续插入；如果要插入20个元素，那么插入11元素时扩容为15，插入第16个元素时，才扩容为15*15=22。
ArrayList(int capacity);

//创建一个包含collection的ArrayList
ArrayList(Collection<? extends E> collection);
```

4.ArrayList API
-----------------------
```
// Collection中定义的API
boolean             add(E object)
boolean             addAll(Collection<? extends E> collection)
void                clear()
boolean             contains(Object object)
boolean             containsAll(Collection<?> collection)
boolean             equals(Object object)
int                 hashCode()
boolean             isEmpty()
Iterator<E>         iterator()
boolean             remove(Object object)
boolean             removeAll(Collection<?> collection)
boolean             retainAll(Collection<?> collection)
int                 size()
<T> T[]             toArray(T[] array)
Object[]            toArray()
// AbstractCollection中定义的API
void                add(int location, E object)
boolean             addAll(int location, Collection<? extends E> collection)
E                   get(int location)
int                 indexOf(Object object)
int                 lastIndexOf(Object object)
ListIterator<E>     listIterator(int location)
ListIterator<E>     listIterator()
E                   remove(int location)
E                   set(int location, E object)
List<E>             subList(int start, int end)
// ArrayList新增的API
Object               clone()
void                 ensureCapacity(int minimumCapacity)
void                 trimToSize()
void                 removeRange(int fromIndex, int toIndex)
```


4.ArrayList的遍历
---------------------
1）通过Iterator遍历
```
Integer value = null;
Iterator iter = list.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```

2）通过随机访问（下标）
```
Integer value = null;
int size = list.size();
for (int i=0; i<size; i++) {
    value = (Integer)list.get(i);        
}
```

3）通过for循环访问
```
Integer value = null;
for (Integer integ:list) {
    value = integ;
}
```

通过测试发现：**遍历ArrayList时，使用随机访问(即，通过索引序号访问)效率最高，而使用迭代器的效率最低！**

参考：https://www.cnblogs.com/skywang12345/p/3308556.html
