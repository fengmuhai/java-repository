java集合类简介
====================

java集合类在实际开发中经常用到，以下结构图展示了java集合类的接口、抽象类、实现类之间的继承和实现关系：

![image](https://github.com/fengmuhai/Hello_Github/blob/master/images/java.collection/collection.jpg)


上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等，
折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等，
而点线边框的是接口，比如Collection，Iterator，List等。

发现一个特点，上述所有的集合类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含hashNext(), next(), remove()三种方法。

它的一个子接口LinkedIterator在它的基础上又添加了三种方法，分别是add()，previous()，hasPrevious()。也就是说如果是先Iterator接口，
那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会在遍历到，通常无序集合实现的都是这个接口，比如HashSet，HashMap；

而那些元素有序的集合，实现的一般都是LinkedIterator接口，实现这个接口的集合可以双向遍历，既可以通过next()访问下一个元素，
又可以通过previous()访问前一个元素，比如ArrayList。
