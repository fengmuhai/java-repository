HashMap简介
==================

1.简述
------------------

哈希表（hash table）也叫散列表，是一种非常重要的数据结构。在讨论哈希表之前，我们先大概了解下其他数据结构在新增，查找等基础操作执行性能。

**数组**：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

**线性链表**：对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

**二叉树**：对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

**哈希表**：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)，接下来我们就来看看哈希表是如何实现达到惊艳的常数阶O(1)的。

　　我们知道，数据结构的物理存储结构只有两种：**顺序存储结构和链式存储结构**（像栈，队列，树，图等是从逻辑结构去抽象的，映射到内存中，也这两种物理组织形式），而在上面我们提到过，在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的主干就是数组。
  

2.底层数据结构
--------------------
在JDK1.6和1.7中，HashMap采用位桶+链表实现，即使用链表处理冲突,同一hash值的键值对会被放在同一个位桶里，当桶中元素较多时，通过key值查找的效率较低。

而JDK1.8中，HashMap采用位桶+链表+红黑树实现，当链表长度超过阈值（8）,时，将链表转换为红黑树，这样大大减少了查找时间。


3.HashMap实现原理
--------------------
HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。
```
//HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂，至于为什么这么做，后面会有详细分析。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
我们向 HashMap 中所放置的对象实际上是存储在该数组当中；

而Map中的key，value则以Entry的形式存放在数组中
```
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
        ...
}        
```

而这个Entry应该放在数组的哪一个位置上（这个位置通常称为位桶或者hash桶，即hash值相同的Entry会放在同一位置，用链表相连），是通过key的hashCode来计算的。
```
final int hash(Object k) {
        int h = 0;
        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
}
```

通过hash计算出来的值将会使用indexFor方法找到它应该所在的table下标：
```
static int indexFor(int h, int length) {
        return h & (length-1);//length=2 的整数次幂
}
```

这个方法其实相当于对table.length取模，**这里哈希值与上(length-1)，length=传入的容量是16的话，16-1=15，二进制1111，即对h取低四位，从而对应0-15个位桶。**

即无论我们指定的容量为多少，构造方法都会将实际容量设为不小于指定容量的2的次方的一个数，且最大值不能超过2的30次方

当两个key通过hashCode计算相同时，则发生了hash冲突(碰撞)，HashMap解决hash冲突的方式是用链表。

当发生hash冲突时，则将存放在数组中的Entry设置为新值的next（这里要注意的是，比如A和B都hash后都映射到下标i中，之前已经有A了，当map.put(B)时，将B放到下标i中，A则为B的next，所以新值存放在数组中，旧值在新值的链表上）

当hash冲突很多时，HashMap退化成链表。

总结一下map.put后的过程：

当向 HashMap 中 put 一对键值时，它会根据 key的 hashCode 值计算出一个位置， 该位置就是此对象准备往数组中存放的位置。


当size大于threshold时，会发生扩容。 threshold等于capacity*load factor
```
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
}
```
jdk7中resize，只有当 size>=threshold并且 table中的那个槽中已经有Entry时，才会发生resize。即有可能虽然size>=threshold，但是必须等到每个槽都至少有一个Entry时，才会扩容。(这句话有待验证)
