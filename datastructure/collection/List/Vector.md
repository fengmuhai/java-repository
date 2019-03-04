Vector简介
=====================

1.简述
---------------------

**Vector 类可以实现可增长的对象数组。**与数组一样，它可以使用整数索引进行访问。但是，**Vector 的大小可以根据需要增大或缩小，**以适应创建 Vector 后进行添加或移除项的操作。**Vector 是同步的，可用于多线程。**


1）Vector 继承了AbstractList，实现了List；所以，它是一个队列，支持相关的添加、删除、修改、遍历等功能。

2）Vector实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在Vector中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。

3）Vector 实现了Cloneable接口，即实现clone()函数。它能被克隆。

4）Vector 实现Serializable接口，支持序列化。


2.Vector的继承关系
----------------------
```
java.lang.Object
  继承者 java.util.AbstractCollection<E>
      继承者 java.util.AbstractList<E>
          继承者 java.util.Vector<E>
所有已实现的接口：
Serializable, Cloneable, Iterable<E>, Collection<E>, List<E>, RandomAccess
直接已知子类：
Stack
```

3.Vector的API
-----------------------
```
synchronized boolean        add(E object)
             void           add(int location, E object)
synchronized boolean        addAll(Collection<? extends E> collection)
synchronized boolean        addAll(int location, Collection<? extends E> collection)
synchronized void           addElement(E object)
synchronized int            capacity()
             void           clear()
synchronized Object         clone()
             boolean        contains(Object object)
synchronized boolean        containsAll(Collection<?> collection)
synchronized void           copyInto(Object[] elements)
synchronized E              elementAt(int location)
             Enumeration<E> elements()
synchronized void           ensureCapacity(int minimumCapacity)
synchronized boolean        equals(Object object)
synchronized E              firstElement()
             E              get(int location)
synchronized int            hashCode()
synchronized int            indexOf(Object object, int location)
             int            indexOf(Object object)
synchronized void           insertElementAt(E object, int location)
synchronized boolean        isEmpty()
synchronized E              lastElement()
synchronized int            lastIndexOf(Object object, int location)
synchronized int            lastIndexOf(Object object)
synchronized E              remove(int location)
             boolean        remove(Object object)
synchronized boolean        removeAll(Collection<?> collection)
synchronized void           removeAllElements()
synchronized boolean        removeElement(Object object)
synchronized void           removeElementAt(int location)
synchronized boolean        retainAll(Collection<?> collection)
synchronized E              set(int location, E object)
synchronized void           setElementAt(E object, int location)
synchronized void           setSize(int length)
synchronized int            size()
synchronized List<E>        subList(int start, int end)
synchronized <T> T[]        toArray(T[] contents)
synchronized Object[]       toArray()
synchronized String         toString()
synchronized void           trimToSize()
```


4.Vector源码分析
----------------------
```
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     * 存储向量组件的数组缓冲区。
     */
    protected Object[] elementData;

    /**
     * Vector 对象中的有效组件数。
     */
    protected int elementCount;

    /**
     * 向量的大小大于其容量时，容量自动增加的量。
     * 即 容量增长系数    
     * @serial
     */
    protected int capacityIncrement;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -2767605614048989439L;

    /**
     * Constructs an empty vector with the specified initial capacity and
     * capacity increment.
     *
     * 使用指定的初始容量和容量增量构造一个空的向量。
     * 指定Vector"容量大小"和"增长系数"的构造函数  
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * Constructs an empty vector with the specified initial capacity and
     * with its capacity increment equal to zero.
     *
     * 使用指定的初始容量和等于零的容量增量构造一个空向量。
     
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    /**
     * Constructs an empty vector so that its internal data array
     * has size {@code 10} and its standard capacity increment is
     * zero.
     * 构造一个空向量，使其内部数据数组的大小为 10，其标准容量增量为零。
     */
    public Vector() {
        this(10);
    }

    /**
     * Constructs a vector containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * 构造一个包含指定 collection 中的元素的向量，
     * 这些元素按其 collection 的迭代器返回元素的顺序排列。
     *
     */
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }

    /**
     * Copies the components of this vector into the specified array.
     * The item at index {@code k} in this vector is copied into
     * component {@code k} of {@code anArray}.
     * 将此向量的组件复制到指定的数组中。此向量中索引 k 处的项将复制到 anArray 的组件 k 中。
     * 将数组Vector的全部元素都拷贝到数组anArray中  
     */
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }

    /**
     * 将当前容量值设为 = 实际元素个数
     * 对此向量的容量进行微调，使其等于向量的当前大小。  
     */
    public synchronized void trimToSize() {
        modCount++; //Vector的改变统计数+1 
        int oldCapacity = elementData.length;
        if (elementCount < oldCapacity) {
            elementData = Arrays.copyOf(elementData, elementCount);
        }
    }

    /**
     * 增加此向量的容量（如有必要），以确保其至少能够保存最小容量参数指定的组件数。
     *
     * @param  minCapacity the desired minimum capacity
     *          minCapacity 所需的最小容量
     */       
    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            modCount++;
            ensureCapacityHelper(minCapacity);//确认“Vector容量”的帮助函数 
        }
    }

    /**
     * 这实现了ensureCapacity的不同步语义。
     * 此类中的同步方法可以在内部调用此方法以确保容量，而不会导致额外同步的成本。
     *
     */
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     *最大值 -8 ，防止OutOfMemoryError
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    //Vector容量是否增加。
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //hugeCapacity 巨大容量 最大容量 Integer.MAX_VALUE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     * Sets the size of this vector. If the new size is greater than the
     * current size, new {@code null} items are added to the end of
     * the vector. If the new size is less than the current size, all
     * components at index {@code newSize} and greater are discarded.
     * 设置此向量的大小。如果新大小大于当前大小，则会在向量的末尾添加相应数量的 null 项。
     * 如果新大小小于当前大小，则丢弃索引 newSize 处及其之后的所有项。
     * @param  newSize   the new size of this vector
     * @throws ArrayIndexOutOfBoundsException if the new size is negative 负数的话，抛出异常
     */
    public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }

    /**
     * 返回此向量的当前容量。 若新初始化
     * Vector<String> v = new Vector<String>(); 
     * v.capacity 返回为10
     * v.size 返回为 0
     */
    public synchronized int capacity() {
        return elementData.length;
    }

    /**
     * Returns the number of components in this vector.
     * 返回此向量中的组件数。 Vector中数组的元素大小！
     * @return  the number of components in this vector
     */
    public synchronized int size() {
        return elementCount;
    }

    /**
     * Tests if this vector has no components.
     * 测试此向量是否不包含组件（元素）
     * 当且仅当此向量没有组件(元素)（也就是说其大小为零）时返回 true；否则返回 false。
     */
    public synchronized boolean isEmpty() {
        return elementCount == 0;
    }

    /**
     * 返回此向量的组件的枚举。返回的 Enumeration 对象将生成此向量中的所有项。
     * 生成的第一项为索引 0 处的项，然后是索引 1 处的项，依此类推。
     * (1)
     * for(Enumeration<String> elements = v.elements();elements.hasMoreElements() ;)
     * System.out.printf(elements.nextElement());
     * (2)
     * while(elements.hasMoreElements())
     * System.out.printf(elements.nextElement());    
     */
    public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");
            }
        };
    }

    /**
     * 如果此向量包含指定的元素，则返回 true。
     * 更确切地讲，当且仅当此向量至少包含一个满足 (o==null ? e==null : o.equals(e)) 的元素 e 时，
     * 返回 true。
     *
     */
    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }

    /**
     * 返回此向量中第一次出现的指定元素的索引，如果此向量不包含该元素，则返回 -1。
     * 更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最低索引 i；
     * 如果没有这样的索引，则返回 -1。
     */
    public int indexOf(Object o) {
        return indexOf(o, 0);
    }

    /**
     * 返回此向量中第一次出现的指定元素的索引，从 index 处正向搜索，
     * 如果未找到该元素，则返回 -1。更确切地讲，
     * 返回满足 (i >= index && (o==null ? get(i)==null : o.equals(get(i)))) 的最低索引 i；
     * 如果没有这样的索引，则返回 -1。
     */
    public synchronized int indexOf(Object o, int index) {
        //分为null和不为null
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)//从index处正向搜索，默认从索引为0处开始
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此向量中最后一次出现的指定元素的索引；如果此向量不包含该元素，则返回 -1。
     * 更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最高索引 i；
     * 如果没有这样的索引，则返回 -1。
     */
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }

    /**
     * 返回此向量中最后一次出现的指定元素的索引，从 index 处逆向搜索，如果未找到该元素，则返回 -1。
     * 更确切地讲，返回满足 (i <= index && (o==null ? get(i)==null : o.equals(get(i)))) 的最高索引 i；
     * 如果没有这样的索引，则返回 -1。
     */
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)//从index处正向搜索，默认从索引为elementCount-1处开始
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回指定索引处的组件。
     * 此方法的功能与 get(int) 方法的功能完全相同（后者是 List 接口的一部分）
     */
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);//按照下标去查找元素
    }

    /**
     * 返回此向量的第一个组件（位于索引 0) 处的项）。
     */
    public synchronized E firstElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(0);
    }

    /**
     * 向量的最后一个组件，即索引 size() - 1 处的组件。
     */
    public synchronized E lastElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(elementCount - 1);
    }

    /**
     * 将此向量指定 index 处的组件设置为指定的对象。丢弃该位置以前的组件(元素)。
     * 索引必须为一个大于等于 0 且小于向量当前大小的值。
     */
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }

    /**
     * 删除指定索引处的组件。此向量中的每个索引大于等于指定 index 的组件都将下移，
     * 使其索引值变成比以前小 1 的值。此向量的大小将减 1。
     * 
     * 索引必须为一个大于等于 0 且小于向量当前大小的值。
     */
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work  让gc做它的工作*/
    }

    /**
     * 将指定对象作为此向量中的组件插入到指定的 index 处。
     * 此向量中的每个索引大于等于指定 index 的组件都将向上移位，使其索引值变成比以前大 1 的值
     */
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }

    /**
     * 将指定的组件添加到此向量的末尾，将其大小增加 1。
     * 如果向量的大小比容量大，则增大其容量。
     */
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);//判读容量大小，是否需要增容
        elementData[elementCount++] = obj; //默认添加
    }

    /**
     * 从此向量中移除变量的第一个（索引最小的）匹配项。
     * 如果在此向量中找到该对象，那么向量中索引大于等于该对象索引的每个组件都会下移，
     * 使其索引值变成比以前小 1 的值
     */
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj); //查询obj索引位置 
        if (i >= 0) {
            removeElementAt(i); //移除变量的第一个匹配项
            return true;
        }
        return false;
    }

    /**
     * 从此向量中移除全部组件，并将其大小设置为零。
     */
    public synchronized void removeAllElements() {
        modCount++;
        // Let gc do its work
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;  //全部设置为 null

        elementCount = 0;  //  elementCount大小设置为 0
    }

    /**
     * 返回向量的一个副本。副本中将包含一个对内部数据数组副本的引用，
     * 而非对此 Vector 对象的原始内部数据数组的引用。
     */
    public synchronized Object clone() {
        try {
            @SuppressWarnings("unchecked")
                Vector<E> v = (Vector<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, elementCount);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError();
        }
    }

    /**
     * Returns an array containing all of the elements in this Vector
     * in the correct order.
     * 返回一个数组，包含此向量中以恰当顺序存放的所有元素。
     * @since 1.2
     */
    public synchronized Object[] toArray() {
        return Arrays.copyOf(elementData, elementCount);
    }

    /**
     * 返回一个数组，包含此向量中以恰当顺序存放的所有元素；返回数组的运行时类型为指定数组的类型。
     */
    @SuppressWarnings("unchecked")
    public synchronized <T> T[] toArray(T[] a) {
        if (a.length < elementCount)
            return (T[]) Arrays.copyOf(elementData, elementCount, a.getClass());

        System.arraycopy(elementData, 0, a, 0, elementCount);

        if (a.length > elementCount)
            a[elementCount] = null;

        return a;
    }

    // Positional Access Operations
    // 定位访问操作
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回向量中指定位置的元素。
     */
    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此向量中指定位置处的元素。
     */
    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    /**
     * 将指定元素添加到此向量的末尾。
     */
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

    /**
     * 移除此向量中指定元素的第一个匹配项，如果向量不包含该元素，则元素保持不变。
     * 更确切地讲，移除其索引 i 满足 (o==null ? get(i)==null : o.equals(get(i))) 的元素（如果存在这样的元素）。
     */
    public boolean remove(Object o) {
        return removeElement(o);
    }

    /**
     * 在此向量的指定位置插入指定的元素。将当前位于该位置的元素（如果有）
     * 及所有后续元素右移（将其索引加 1）。
     */
    public void add(int index, E element) {
        insertElementAt(element, index);
    }

    /**
     * 移除此向量中指定位置的元素。将所有后续元素左移（将其索引减 1）。
     * 返回此向量中移除的元素。
     */
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
    }

    /**
     * 从此向量中移除所有元素。此调用返回后，向量将为空（除非抛出了异常）。
     */
    public void clear() {
        removeAllElements();
    }

    // Bulk Operations
    // 批量操作

    /**
     * 如果此向量包含指定 Collection 中的所有元素，则返回 true。
     */
    public synchronized boolean containsAll(Collection<?> c) {
        return super.containsAll(c);
    }

    /**
     * 将指定 Collection 中的所有元素添加到此向量的末尾，按照指定 collection 的迭代器所返回的顺序添加这些元素。
     *
     */
    public synchronized boolean addAll(Collection<? extends E> c) {
        modCount++;
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);
        System.arraycopy(a, 0, elementData, elementCount, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    /**
     * 从此向量中移除包含在指定 Collection 中的所有元素。
     */
    public synchronized boolean removeAll(Collection<?> c) {
        return super.removeAll(c);
    }

    /**
     * 在此向量中仅保留包含在指定 Collection 中的元素。
     * 换句话说，从此向量中移除所有未包含在指定 Collection 中的元素。
     */
    public synchronized boolean retainAll(Collection<?> c) {
        return super.retainAll(c);
    }

    /**
     * 在指定位置将指定 Collection 中的所有元素插入到此向量中。
     * 将当前位于该位置的元素（如果有）及所有后续元素右移（增大其索引值）。
     */
    public synchronized boolean addAll(int index, Collection<? extends E> c) {
        modCount++;
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    /**
     * 比较指定对象与此向量的相等性。
     */
    public synchronized boolean equals(Object o) {
        return super.equals(o);
    }

    /**
     * 返回此向量的哈希码值。
     */
    public synchronized int hashCode() {
        return super.hashCode();
    }

    /**
     * 返回此向量的字符串表示形式，其中包含每个元素的 String 表示形式。
     */
    public synchronized String toString() {
        return super.toString();
    }

    /**
     * 返回此 List 的部分视图，元素范围为从 fromIndex（包括）到 toIndex（不包括）。
     */
    public synchronized List<E> subList(int fromIndex, int toIndex) {
        return Collections.synchronizedList(super.subList(fromIndex, toIndex),
                                            this);
    }

    /**
     * 从此 List 中移除其索引位于 fromIndex（包括）与 toIndex（不包括）之间的所有元素
     */
    protected synchronized void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = elementCount - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // Let gc do its work
        int newElementCount = elementCount - (toIndex-fromIndex);
        while (elementCount != newElementCount)
            elementData[--elementCount] = null;
    }

    /**
     * 将Vector实例的状态保存到流（即，序列化它）。 此方法执行同步以确保序列化数据的一致性。
     */
    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
        final java.io.ObjectOutputStream.PutField fields = s.putFields();
        final Object[] data;
        synchronized (this) {
            fields.put("capacityIncrement", capacityIncrement);
            fields.put("elementCount", elementCount);
            data = elementData.clone();
        }
        fields.put("elementData", data);
        s.writeFields();
    }

    /**
     * 对列表中的元素返回一个列表迭代器（以正确的顺序），
     * 从列表中指定的位置开始。 指定的索引指示由初始调用返回到next的第一个元素。 
     * 对上一个的初始调用将返回具有指定索引减1的元素。
     * 
     *返回的列表迭代器是fail-fast的。
     *
     */
    public synchronized ListIterator<E> listIterator(int index) {
        if (index < 0 || index > elementCount)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 返回此列表中的元素（按正确顺序）的列表迭代器。
     *
     * 返回的列表迭代器是fail-fast的。
     *
     */
    public synchronized ListIterator<E> listIterator() {
        return new ListItr(0); //ListItr extends Itr implements ListIterator<E>
    }

    /**
     * 返回此列表中的元素（按正确顺序）的列表迭代器。
     *
     * 返回的列表迭代器是fail-fast的。
     */
    public synchronized Iterator<E> iterator() {
        return new Itr(); //class Itr implements Iterator<E>
    }

    /**
     * An optimized version of AbstractList.Itr
     * 这个下面不在分析，大概实现了要Iterator ，实现具体的一些方法
     */
    private class Itr implements Iterator<E> {
        
    }

    
}
```


5.小结
--------------------
1） Vector实际上是通过一个数组去保存数据的。当我们构造Vecotr时；若使用默认构造函数，则Vector的默认容量大小是10。

2）当Vector容量不足以容纳全部元素时，Vector的容量会增加。若容量增加系数 大于0，则将容量的值增加“容量增加系数”；否则，将容量大小增加一倍。

3） Vector的克隆函数，即是将全部元素克隆到一个数组中。

4） 很多方法都加入了synchronized同步语句，来保证线程安全。

5） 同样在查找给定元素索引值等的方法中，源码都将该元素的值分为null和不为null两种情况处理，Vector中也允许元素为null。

6） 遍历Vector，使用索引的随机访问方式最快，使用迭代器最慢。

7） Vector很多地方都与ArrayList实现大同小异，现在已经基本不再使用。

