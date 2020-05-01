

本文从源码角度来梳理一下`Java7 HashMap`，主要从以下三个方面展开：  

- `存储结构`  
- `put()`  
- `get()`  


## 存储结构  

先来看下`HashMap`的存储结构：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/collection_java7_HashMap_structure.png)  

上图来自`JavaDoop博主`，原文请参考：  

[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://www.javadoop.com/post/hashmap){target="_blank"}  

可以看到`HashMap`本质就是一个`数组`，其中的每个元素都是一个`单向链表`。  

再来看三个重要的参数：  

- `capacity`：当前数组的容量，始终保持`2^n`，可以先考虑下为什么？  
- `loadFactor`：负载因子，其实就是一个系数，默认值为`0.75`。  
- `threshold`：扩容的阈值，等于`capacity * loadFactor`。  


## put()  

然后来说下如何插入键值对：  

``` java
public V put(K key, V value) {
    // Part 1
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // Part 2
    if (key == null)
        return putForNullKey(value);
    // Part 3
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    // Part 4
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

我们可以将其划分为四个部分：  

- `Part 1`：初始化数组。  
- `Part 2`：插入`Key`为`null`的键值对。  
- `Part 3`：确认桶下标，并可能替换并返回旧值。  
- `Part 4`：插入新键值对。  


#### 初始化数组  

``` java
private void inflateTable(int toSize) {
    // Find a power of 2 >= toSize
    int capacity = roundUpToPowerOf2(toSize);

    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}

private static int roundUpToPowerOf2(int number) {
    // assert number >= 0 : "number must be non-negative";
    return number >= MAXIMUM_CAPACITY
            ? MAXIMUM_CAPACITY
            : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```

前面提到，`capacity`容量的值一定是`2^n`，那么如何保证这一点？  

这就需要用到`roundUpToPowerOf2()`方法。  

举个例子，当`toSize = 7`。此时：`number - 1 = 6, 6 << 1 = 12`。  

所以`Integer.highestOneBit((number - 1) << 1) = 8`  

因此`capacity = 8 = 2 ^ 3`。  

确定容量后，就可以创建并初始化数组了：`table = new Entry[capacity];`  


#### 插入`Key`为`null`的键值对  

再来看看`HashMap`如何插入`Key`为`null`的键值对：  

``` java
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

可以看到，`Key`为`null`的键值对只会保存在下标为`0`的桶中。  


接下来我会分为三种情况来分析这个方法：  

- 第一种： 往一个新的`HashMap`中插入`{null: "1"}`。  
- 第二种： 在第一种的基础上，继续插入`{null: "2"}`。  
- 第三种： 下标为`0`的桶中正好保存着其他键值对，此时我们继续插入`{null: "3"}`。  

先来说第一种：  

``` java
private static void justTest() {
    HashMap<String, String> map = new HashMap<>(2);
    map.put(null, "1");
}

/* HashMap line 486 */
public V put(K key, V value) {
    ...
    if (key == null)
        return putForNullKey(value);
    ...
}

/* HashMap line 512 */
private V putForNullKey(V value) {
    ...
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}

/* HashMap line 877 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    ...
    createEntry(hash, key, value, bucketIndex);
}

/* HashMap line 895 */
void createEntry(int hash, K key, V value, int bucketIndex) {
    // e = null
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

Entry(int h, K k, V v, Entry<K,V> n) {
    value = v;
    next = n;
    key = k;
    hash = h;
}
```

关键在于`createEntry()`这个方法，直接看下面的示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/put_key_null_1.png)  


再来看第二步，在上面的基础上，继续插入`{null: "2"}`：  

``` java
private static void justTest() {
    HashMap<String, String> map = new HashMap<>(2);
    map.put(null, "1");
    map.put(null, "2");
}

/* HashMap line 512 */
private V putForNullKey(V value) {
    // enter for loop.
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    ...
}
```

这次不一样了，进入`for`循环，由于在`index = 0`的桶中已存在`{null: "1"}`，因此进行`Value`替换，并返回旧值`1`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/put_key_null_2.png)  


再来看最后一种情况：`index = 0`的桶中已存在其他键值对，此时插入`{null: "3"}`：  

``` java
private static void justTest() {
    HashMap<String, String> map = new HashMap<>(2);
    map.put("K1", "V1");
    map.put(null, "3");
}

/* HashMap line 512 */
private V putForNullKey(V value) {
    ...
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
...
```

继续追踪下去，先不考虑扩容，结果应该是这样的：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/put_key_null_3.png)  



#### 确认桶下标  

接下来让我们再来看下如何确定桶下标？  

``` java
public V put(K key, V value) {
    ...
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    ...
}

/* HashMap line 374 */
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

本质就是用`hash`值对数组容量`取余`！  

但是为了提升计算效率，所以使用`按位与`运算，这也回答了上面的问题：容量的大小为什么要是`2^n`。    

举个例子： `hash = 100, capacity = length = 16`。  

`index = 100 % 16 = 4 = 100 & 15`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/index_for.png)  


#### 插入新键值对  

最后让我们再来看下如何插入新键值对：  

``` java
/* HashMap line 486 */
public V put(K key, V value) {
    ...
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

/* HashMap line 877 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}
```

可以看到，其实分为两步：`扩容`和`插入`。  

插入过程上面已经讲过了，这里我们来介绍下如何扩容？  

举个例子：  

``` java
private static void resizeTest() {
    HashMap<String, String> map = new HashMap<>(2);
    map.put("K1", "V1");
    map.put("K2", "V2");
    map.put("K3", "V3");
}

/* HashMap line 877 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    // size = 2, threshold = 1
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    ...
}

/* HashMap line 572 */
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

/* HashMap line 589 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

在`resize()`方法中，首先创建了一个`双倍`容量的数组，然后通过`transfer()`方法复制原先数组中的数据，这里需要格外注意一点，在并发情况下，`transfer()`这个方法可能会造成循环链表，从而导致后续查询过程发生死循环！  

具体可参考： [A Beautiful Race Condition](https://mailinator.blogspot.com/2009/06/beautiful-race-condition.html){target="_blank"}  


后面就是重新计算`hash`值和桶下标，最后插入键值对。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/put_resize.png)  


最后分享一个使用`put()`方法的小技巧，有兴趣的可以看看：  

[leetcode - [205] Isomorphic Strings](http://yaoyichen.cn/algorithm/2020/04/21/leetcode-205.html){target="_blank"}  

`Amazing!`  

## get()  

再来讲下如何从`HashMap`中获取数据：  


``` java
private static void getTest() {
    HashMap<String, String> map = new HashMap<>(2);
    map.put("K1", "V1");
    map.put("K2", "V2");
    map.put("K3", "V3");
    String k2 = map.get("K2");
}

/* HashMap line 414 */
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}

/* HashMap line 457 */
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

计算`Key`的`hash`值，然后计算桶下表，比较`Key`值，返回`Value`或`null`。  


## `Reference`  
- [Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://www.javadoop.com/post/hashmap){target="_blank"}  
- [A Beautiful Race Condition](https://mailinator.blogspot.com/2009/06/beautiful-race-condition.html){target="_blank"}  