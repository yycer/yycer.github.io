---
layout: post
title: Java - [容器]List
category: collection
tags: [spring, java]
excerpt: Java Collection List
---

本文主要从`存储结构`、`扩容方法`、`常用方法`等角度分析以下四种`List`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/list_frame.png)  

> 注意： 全文源码均出自`JDK8`。  

## `ArrayList`  

先从五个维度梳理下`ArrayList`的知识点：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/arraylist_frame.png)  


#### 1、 存储结构  

``` java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;

/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

可以看到它的数据结构就是一个`Object`类型的数组。  

当使用无参构造函数时，`elementData`是一个类型为`Object`的空数组。  

再来区分下`capacity[数组容量]`和`size[数组实际大小]`。  

举个例子：  

``` java
@Test
public void arrayListTest(){
    ArrayList<String> al = new ArrayList<>(11);
    al.add("yyc");
    al.add("frankie");
}
```

此时，`capacity[数组容量] = 11`，如下图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/arraylist_elementdata.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/arraylist_elementdata_length.png)  

而`size[数组实际大小] = 2`，因为它统计的是数组中实际填充元素的个数：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/arraylist_size_method.png)  


#### 2、 扩容  

再来看下`ArrayList`是如何扩容的。  


``` java
@Test
public void resizeTest(){
    ArrayList<String> al = new ArrayList<>(2);
    al.add("yyc");
    al.add("frankie");
    al.add("Jack");
}
```

当添加`Jack`时，发生了什么？  

``` java
/* ArrayList line 496 */
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}
```

此时，`size = 2`，`elementData`如下所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/resize_elementdata.png)  


``` java
/* ArrayList line 483 */
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

/* ArrayList line 241 */
private Object[] grow() {
    return grow(size + 1);
}

    }
/* ArrayList line 236 */
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

/* ArrayList line 254 */
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    // oldCapacity = 2
    int oldCapacity = elementData.length;
    // newCapacity = 2 + 1 = 3
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // newCapacity - minCapacity = 3 - 3 = 0
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        // return 3
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}
```

总结一下扩容流程：  

- 创建一个新的`Object`类型的数组，其容量为原数组的`1.5`倍。  
- 使用新数组替换老数组。  

#### 3、 删除元素  

大家有没有考虑过从`ArrayList`中删除一个元素，其内部到底发生了什么？  

主要分为两种情况，若删除的元素是尾元素，那么直接将尾元素指向`null`。  

否则，需要将`[i+1, len-1]`的所有元素，全部往前移一位，最后将尾元素指向`null`。  


``` java
@Test
public void deleteElementTest(){
    ArrayList<String> al = new ArrayList<>(5);
    al.add("1");
    al.add("2");
    al.add("3");
    al.add("4");
    al.add("5");
    al.remove("2");
}
```

先来看下图释：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/arraylist_remove.png)  

再来追踪下源码：  

``` java
/* ArrayList line 644 */
public boolean remove(Object o) {
    final Object[] es = elementData;
    // size = 5
    final int size = this.size;
    int i = 0;
    found: {
        if (o == null) {
            for (; i < size; i++)
                if (es[i] == null)
                    break found;
        } else {
            for (; i < size; i++)
                if (o.equals(es[i]))
                    break found;
        }
        return false;
    }
    // i = 1
    fastRemove(es, i);
    return true;
}

/* ArrayList line 668 */
private void fastRemove(Object[] es, int i) {
    modCount++;
    final int newSize;
    // newSize = size - 1 = 5 - 1 = 4
    if ((newSize = size - 1) > i)
        System.arraycopy(es, i + 1, es, i, newSize - i);
    es[size = newSize] = null;
}

/**
  * System line 531
  * @param      src      the source array.
  * @param      srcPos   starting position in the source array.
  * @param      dest     the destination array.
  * @param      destPos  starting position in the destination data.
  * @param      length   the number of array elements to be copied.
  */
@HotSpotIntrinsicCandidate
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

所以，`ArrayList`一旦发生扩容或删除元素，就会造成一定的性能消耗。  


#### 4、 序列化  

有没有注意到`elementData`前面加了一个关键字 `transient`？  

``` java
transient Object[] elementData; // non-private to simplify nested class access
```

那么这个关键字的作用是什么？  

> 当你有些字段不想被序列化，就可以使用`transient`关键字。  

`ArrayList`实现了`writeObject()`和`readObject()`来控制只序列化数组中已填充的元素。  

有兴趣的同学，可以翻下这个两个方法的源码。  


#### 5、 `Fail-Fast`  

首先需要提下`modCount`字段，它的作用是什么呢？  

> 用来记录`ArrayList`结构发生变化的次数。  

那么哪些行为才会使结构发生变化？  

> 比如：插入、删除一个元素。需要注意，仅仅是设置元素的值并不算结构发生改变。  

在进行`序列化`或`迭代`等操作时，需要比较操作前后的`modCount`字段是否发生改变，如果改变了就会抛出`ConcurrentModificationException`。  
    

## `Vector`  

然后从四个角度来介绍下`Vector`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/vector_frame.png)  

#### 1、 同步  

在方法实现上，`Vector`与`ArrayList`很类似，区别就在于前者使用了同步方法`synchronized`。  

``` java
public synchronized boolean add(E e) {
    modCount++;
    add(e, elementData, elementCount);
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```


#### 2、 扩容  

``` java
/* Vector line 276 */
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    ...
}
```

若`capacityIncrement > 0`，则`newCapacity = oldCapacity + capacityIncrement`。  

否则`newCapacity = oldCapacity + oldCapacity`。  

那么`capacityIncrement`如何设置呢？  


``` java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

#### 3、 与ArrayList的区别？  

从以下两个维度分析一下：  

- `时间消耗`：由于`Vector`是同步的，时间消耗会比`ArrayList`长，因此访问速度更慢。  
- `扩容大小`：`Vector`每次扩容都是翻倍(也可以通过构造函数指定增量)，而`ArrayList`是`1.5`倍。  

#### 4、 替代方案  

除了使用`Vector`，我们还有哪些方案可以获得一个线程安全的`ArrayList`？  

- 第一种：使用`Collections.synchronizedList()`方法  

``` java
/* Collections line 2383 */
public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list) :
            new SynchronizedList<>(list));
}
```


- 第二种：使用`CopyOnWriteArrayList`读写分离类。  

``` java
package java.util.concurrent;

/**
  * A thread-safe variant of {java.util.ArrayList} in which all mutative
  * operations ({add}, {set}, and so on) are implemented by
  * making a fresh copy of the underlying array.
  */
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ...
}
```

## `CopyOnWriteArrayList`  

接着我们从三个维度分析下这个类：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/copyonwritearraylist_frame.png)  

#### 存储结构  

``` java
/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

基本数据结构和`ArrayList`一样，都是`Object`类型的数组。  不过，还有个`volatile`关键字修饰。  

先简单提下`volatile`关键字的作用：  

> 本质在于保证同一变量在不同线程之间的可见性。  


#### 核心优势  

当添加一个元素时，会在一个`监视器锁`内完成如下操作：  

- 获取原始数组  
- 创建一个新的数组，其容器为`oldCapacity + 1`，并将原始数组中的元素移到新数组中。  
- 新数组替换原始数组  

源码如下：  

``` java
/* CopyOnWriteArrayList line 428 */
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}

/* CopyOnWriteArrayList line 110 */
final Object[] getArray() {
    return array;
}

/* CopyOnWriteArrayList line 117 */
final void setArray(Object[] a) {
    array = a;
}
```

当读取元素时，直接从原始数组中获取：  


``` java
/* CopyOnWriteArrayList line 398 */
public E get(int index) {
    return elementAt(getArray(), index);
}

/* CopyOnWriteArrayList line 385 */
static <E> E elementAt(Object[] a, int index) {
    return (E) a[index];
}
```


#### 使用场景  

`CopyOnWriteArrayList`在写操作的同时允许读操作，大大提升读操作的性能，因此非常适合`读多写少`的场景。  

但是也有两点不足：  

- 内存占用： 每次写操作都需要额外创建一个新的数组。  
- 数据不一致： 读操作不能读取实时的数据。  

所以`CopyOnWriteArrayList`不适合`内存敏感`以及`对实时性要求很高`的场景。  



## `LinkedList`  

最后我们从四个方面分析下`LinkedList`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linkedlist_frame.png)  


#### 1、 存储结构  

``` java
/**
 * Pointer to first node.
 */
transient Node<E> first;

/**
 * Pointer to last node.
 */
transient Node<E> last;
```

可以看到，`LinkedList`的数据结构是双向链表。  


#### 2、 添加元素  

再来看下`LinkedList`插入元素的过程：  


``` java
@Test
public void addTest(){
    LinkedList<String> ll = new LinkedList<>();
    boolean ret1 = ll.add("yyc");
    boolean ret2 = ll.add("frankie");
}

/* LinkedList line 341 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/* LinkedList line 144 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

/* LinkedList line 974 */
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

总结一下插入过程：  

- 首先，创建一个新的节点。  
- 然后，找到原始双向链表的尾结点，并令其指向新节点。  

#### 3、 删除元素  



``` java
@Test
public void removeTest(){
    LinkedList<String> ll = new LinkedList<>();
    boolean ret1 = ll.add("yyc");
    boolean ret2 = ll.add("frankie");
    boolean ret3 = ll.add("Jack");
    ll.remove("frankie");
}

/* LinkedList line 395 */
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

/* LinkedList line 213 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

具体可参考下面的图释：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linkedlist_remove.png)  


#### 4、 与ArrayList的区别？  

`ArrayList`基于数组实现，而`LinkedList`基于双向链表实现。  

- `ArrayList`：查询时间复杂度约为`O(1)`，但插入和删除元素代价很高，因为涉及后置元素的复制与扩容，特别是涉及数组前半部分的操作。  
- `LinkedList`：查询时间复杂度为`O(N)`，但插入和删除元素代价很小，只需要调整前后指针。  


## `Reference`  
- [Java 容器](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%AE%B9%E5%99%A8){:target="_blank"}

