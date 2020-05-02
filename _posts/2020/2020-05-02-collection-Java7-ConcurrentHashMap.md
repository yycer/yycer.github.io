---
layout: post
title: Java7 - [容器]ConcurrentHashMap
category: collection
tags: [spring, java]
excerpt: Java7 ConcurrentHashMap
---


本文从源码角度来梳理一下`Java7 ConcurrentHashMap`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/collection-Java7-ConcurrentHashMap-frame.png)  


## 存储结构  

先来看下`JavaDoop博主`画的示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/collection_java7_ConcurrentHashMap_structure.png)  

`ConcurrentHashMap`的基础结构是`Segment[]`数组，`Segment`代表部分、一段的意思，  

因此很多地方会将其描述成`分段锁`，我继续沿用`JavaDoop博主`的说法，将其称之为`槽`。  

> Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 Segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。  

来看`Segment`的内部结构：  

``` java
/**
 * The load factor for the hash table.  Even though this value
 * is same for all segments, it is replicated to avoid needing
 * links to outer object.
 * @serial
 */
final float loadFactor;

/**
 * The table is rehashed when its size exceeds this threshold.
 * (The value of this field is always <tt>(int)(capacity *
 * loadFactor)</tt>.)
 */
transient int threshold;

/**
 * The per-segment table. Elements are accessed via
 * entryAt/setEntryAt providing volatile semantics.
 */
transient volatile HashEntry<K,V>[] table;

Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
    this.loadFactor = lf;
    this.threshold = threshold;
    this.table = tab;
}

static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;
    ...
}
```

可以看到，槽内的内部是一个`HashEntry[]`数组，其中每个元素都是一个单向链表，也可以称之为桶，与`HashMap`结构类似。  

再来讲下几个参数：  

- `loadFactor`： `HashEntry`数组中的`负载因子`，默认值为`0.75`  
- `threshold`： 扩容阈值，等于`capacity * loadFactor`   



## 初始化Segment[0]  

先来看个例子，后面还会用到，暂时称它为`代码片段A`：  

``` java
public static void main(String[] args) {
    ConcurrentHashMap<String, String> chm = new ConcurrentHashMap<>(4);
    chm.put("city2", "sh2");
    chm.put("city3", "sh3");
    chm.put("city13", "sh13");
    chm.put("city14", "sh14");
}
```

先来看`ConcurrentHashMap`的构造器：  

``` java
/* ConcurrentHashMap line 860 */
public ConcurrentHashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

解释下三个参数：  

- `initialCapacity`：初始容量，默认为`16`  
- `DEFAULT_LOAD_FACTOR`：负载因子，上面提到过，默认为`0.75`  
- `DEFAULT_CONCURRENCY_LEVEL`： 并发等级，也就是`Segment`数组的大小，默认`16`  


继续深入：  

``` java
/* ConcurrentHashMap line 800 */
@SuppressWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Part 1
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // Part 2
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // Part 3: create segments and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

主要分为三步：  

- 计算`sshift`和`ssize`  
- 计算`Segment`内部`Table`的容量  
- 创建`Segment`数组，并初始化`Segment[0]`  

我们来具体看下：  

#### 计算sshift和ssize  

``` java
int sshift = 0;
int ssize = 1;
while (ssize < concurrencyLevel) {
    ++sshift;
    ssize <<= 1;
}
this.segmentShift = 32 - sshift;
this.segmentMask = ssize - 1;
if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
```

举个实例，当`concurrencyLevel = 16`，也就是默认值。  

执行完`while`循环后，`sshift = 4, ssize = 16`  

同时，`segmentShift = 32 - 4 = 28, segmentMask = 15`  

我们先将`segmentShift`和`segmentMask`称之为`移位数`和`掩码`。  


#### 计算Segment内部Table的容量  

``` java
int c = initialCapacity / ssize;
if (c * ssize < initialCapacity)
    ++c;
int cap = MIN_SEGMENT_TABLE_CAPACITY;
while (cap < c)
    cap <<= 1;
```

首先，如何确定每个`Segment`被分配多少容量？  

答案是均分：  `int c = initialCapacity / ssize;`  

但是有个限制，该容量最少为`2`，防止添加第一个元素就发生扩容。   


#### 创建Segment数组，并初始化Segment[0]  

``` java
Segment<K,V> s0 =
    new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                     (HashEntry<K,V>[])new HashEntry[cap]);
Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
this.segments = ss;
```

为什么只初始化`Segment[0]`？  

这个问题的答案在后面。  


## put()  

先来看下概况：  

``` java
/* ConcurrentHashMap line 1120 */
@SuppressWarnings("unchecked")
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    // Part 1
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    // Part 2
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    // Part 3
    return s.put(key, hash, value, false);
}
```

我们将其切分成三块：  

- `Part 1`： 计算槽下标  
- `Part 2`： 初始化槽`j`  
- `Part 3`： 插入键值对  


#### 计算槽下标  

``` java
int hash = hash(key);
int j = (hash >>> segmentShift) & segmentMask;
```

这就用到上一步构造器中计算的`位移数`和`掩码`。  


#### 初始化槽`j`  

``` java
if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
    s = ensureSegment(j);

/* ConcurrentHashMap line 736 */
@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // recheck
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

这里有两点需要注意：  

第一，回答了上面的问题： 为什么只初始化`Segment[0]`？  

使用当前`Segment[0]`中的数组长度和负载因子来初始化`Segment[K]`。  

第二，使用`CAS`确保只有一个线程能成功设值。  


#### 插入键值对  

再来看插入键值对这个关键方法：  

``` java
/* ConcurrentHashMap line 431 */
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // Part 1
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        // Part 2
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            // Part 3
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                // Part 4
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}

/* ConcurrentHashMap line 303 */
@SuppressWarnings("unchecked")
static final <K,V> HashEntry<K,V> entryAt(HashEntry<K,V>[] tab, int i) {
    return (tab == null) ? null :
        (HashEntry<K,V>) UNSAFE.getObjectVolatile
        (tab, ((long)i << TSHIFT) + TBASE);
}
```

先要明确一点，当你进入这个整个`put()`方法，就说明你已经进入`Segment`内部，也就是`HashEntry`数组中了。  

老规矩，还是分成四个部分：  

- `Part 1`： 上锁  
- `Part 2`： 确定桶下标  
- `Part 3`： 覆盖值  
- `Part 4`： 新增键值对与扩容  


#### 上锁  

``` java
HashEntry<K,V> node = tryLock() ? null :
    scanAndLockForPut(key, hash, value);

/* ConcurrentHashMap line 386 */
static final int MAX_SCAN_RETRIES =
    Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

/* ConcurrentHashMap line 551 */
private HashEntry<K,V> `scanAndLockForPut`(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

首先尝试通过`tryLock()`方法快速获取该`Segment`的独占锁，若失败，则执行`scanAndLockForPut()`方法。  

该方法有两个出口，一个是`tryLock()`方法执行成功，第二个是重试次数超过`MAX_SCAN_RETRIES`，执行`lock()`方法，阻塞当前线程。 关于`tryLock()`和`lock()`方法的区别可以参考这篇文章：  

[[Java] lock、tryLock和lockInterruptibly的差別](https://pandaforme.github.io/2016/12/09/Java-lock%E3%80%81tryLock%E5%92%8ClockInterruptibly%E7%9A%84%E5%B7%AE%E5%88%A5/){:target="_blank"}  



#### 确定桶下标  

``` java
HashEntry<K,V>[] tab = table;
int index = (tab.length - 1) & hash;
HashEntry<K,V> first = entryAt(tab, index);
```

本质还是通过`取余`获取桶下标。  


#### 覆盖值  

``` java
...
for (HashEntry<K,V> e = first;;) {
    if (e != null) {
        K k;
        if ((k = e.key) == key ||
            (e.hash == hash && key.equals(k))) {
            oldValue = e.value;
            if (!onlyIfAbsent) {
                e.value = value;
                ++modCount;
            }
            break;
        }
        e = e.next;
    }
    ...
```

若想插入的`Key`在当前桶中已存在，且`onlyIfAbsent = false`，则新值替换旧值。  



#### 新增键值对与扩容  

``` java
HashEntry<K,V> node = tryLock() ? null :
    scanAndLockForPut(key, hash, value);
...
for (HashEntry<K,V> e = first;;) {
    else {
        if (node != null)
            node.setNext(first);
        else
            node = new HashEntry<K,V>(hash, key, value, first);
        int c = count + 1;
        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
            rehash(node);
        else
            setEntryAt(tab, index, node);
        ++modCount;
        count = c;
        oldValue = null;
        break;
    }
...

/* ConcurrentHashMap line 314 */
static final <K,V> void setEntryAt(HashEntry<K,V>[] tab, int i,
                                       HashEntry<K,V> e) {
    UNSAFE.putOrderedObject(tab, ((long)i << TSHIFT) + TBASE, e);
}
```

若当前`Segment`成功上锁，就会创建一个新的节点`node`，然后执行`setEntryAt()`方法将节点插入单向链表中。  

最后，介绍下如何扩容`rehash()`：


``` java
/* ConcurrentHashMap line 480 */
@SuppressWarnings("unchecked")
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // Double capacity
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    int sizeMask = newCapacity - 1;
    for (int i = 0; i < oldCapacity ; i++) {
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;
            if (next == null)   //  Single node on list
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                newTable[lastIdx] = lastRun;
                // Clone remaining nodes
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

首先明确一点，新的`HashEntry[]`数组的容量是之前的两倍：  

`int newCapacity = oldCapacity << 1;`  

让我们通过上面`代码片段A`和下面的示意图，理解下整个过程：  

扩容之前：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/rehash_before.png)  

扩容之后：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/rehash_after.png)  


建议读者亲自调试下，感受整个过程。  


## get()    

``` java
/* ConcurrentHashMap line 985 */
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

先定位到`槽`，再定位到`HashEntry`，最后取其值。  


## Reference    
- [Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://www.javadoop.com/post/hashmap){:target="_blank"}  