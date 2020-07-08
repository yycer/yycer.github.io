---
layout: post
title: 并发 - [5] 阻塞队列
category: multithread
tags: [multithread]
excerpt: Concurreny Blocking Queue
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [5]阻塞队列`  


## 阻塞队列有什么特点？  

首先，要知道什么是阻塞队列？  

阻塞队列就是`BlockingQueue`，它是一个继承`Queue`的接口。  

先来介绍一下两个最重要的方法：  

- `put()`  

如果队列还没满，那么正常插入。否则，让插入的线程陷入`阻塞`状态。  

直到队列有了空闲空间，才会通知阻塞的线程，让其执行添加操作。  

- `take()`  

获取并移除队列的头结点，如果队列中有数据，那么正常获取。  

否则，让获取线程陷入`阻塞`状态。直到队列中有数据，才会通知阻塞的线程，让其执行获取操作。

第三个特点：`是否有界`？  

其实就是队列的容量大小，可以分为`无界队列`，如：`LinkedBlockingQueue`。`有界队列`，如：`ArrayBlockingQueue`。  



## 阻塞队列有哪些常用方法？  

针对特殊情况的不同处理方式，可以分为以下三组：  

- 抛出异常型: `add()`, `remove()`, `element()`  
- 返回结果但不抛出异常型：`offer()`, `poll()`, `peek()`  
- 阻塞型: `put()`, `take()`  

阻塞型的两个方法上面已经介绍过了，所以重点来介绍前两组方法：  

第一组：  

- `add()`  

作用： 往`队尾`插入一个元素，如果队列已满，则`抛出异常`。  

``` java
private static ArrayBlockingQueue<Integer> abq = new ArrayBlockingQueue<>(2);

public static void main(String[] args) {
    add();
}

private static void add() {
    abq.add(1);
    abq.add(2);
    abq.add(3);
}
```

结果如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/add_exception.png)  

- `remove()`  

作用： 删除队列`头结点`，如果队列为空，则抛出异常。  

``` java
private static ArrayBlockingQueue<Integer> abq = new ArrayBlockingQueue<>(2);

public static void main(String[] args) {
    remove();
}

private static void remove() {
    abq.add(1);
    abq.add(2);
    System.out.println(abq.remove()); // 1
    System.out.println(abq.remove()); // 2
    System.out.println(abq.remove());
}
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/remove_exception.png)  


- `element()`  

作用： 返回队列`头结点`，如果队列为空，则抛出异常。  



``` java
private static ArrayBlockingQueue<Integer> abq = new ArrayBlockingQueue<>(2);

public static void main(String[] args) {
    element();
}

private static void element() {
    abq.add(1);
    abq.add(2);
    System.out.println(abq.remove()); // 1
    System.out.println(abq.remove()); // 2
    System.out.println(abq.element());
}
```


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/element_exception.png)  


再来看下第二组方法：  

- `offer()`  

作用： 往`队尾`插入一个元素，并用`返回值`提示是否插入成功。  

``` java
private static void offer() {
    System.out.println(abq.offer(1)); // true
    System.out.println(abq.offer(2)); // true
    System.out.println(abq.offer(3)); // false
}
```

- `poll()`  

作用： 删除`队列头`结点，如果队列为空，则返回`null`。  

所以不允许往队列中插入`null`，防止两者无法区分。  


``` java
private static void poll() {
    abq.offer(1);
    abq.offer(2);
    System.out.println(abq.poll()); // 1
    System.out.println(abq.poll()); // 2
    System.out.println(abq.poll()); // null
}
```


- `peek()`  

返回`队列头`结点，如果队列为空，则返回`null`。  


``` java
private static void peek() {
    System.out.println(abq.peek()); // null
}
```


## 有哪几种常见的阻塞队列？  

主要分为以下`五`种：  

### `ArrayBlockingQueue`  

它是最典型的`有界队列`，其内部是用`数组`存储元素的，  

利用`ReentrantLock`实现`线程安全`，创建时需要`指定容量`且之后`不可扩容`。  


### `LinkedBlockingQueue`   

这是一个内部用`链表`实现的`无界`阻塞队列，  

如果我们不指定它的初始容量，那么它的默认值为`Integer.MAX_VALUE`。  


### `SychronousQueue`   

注意：它的`容量为0`，它所做的就是直接传递`[direct handoff]`  

因此每次取数据都要先阻塞，直到有数据被放入，  

同理，每次放数据也会阻塞，直到有消费者来取。  

### `PriorityBlockingQueue`   

它是一个支持`优先级`的`无界`阻塞队列，  

可以通过自定义类实现`compareTo()`方法来指定元素的排序规则，  

也可以在初始化时通过构造器参数`Comparator`来指定排序规则，  

插入队列的对象必须是可比较大小的，也就是`Comparable`的。  

### `DelayQueue`   

它可以设置让队列中的任务`延迟`多久后执行。  

它也是`无界`队列，放入的元素必须实现`Delayed`接口，  

而`Delayed`接口又继承了`Comparable`接口。  



## (非)阻塞队列的并发安全原理是什么？  

阻塞队列主要是利用`ReentrantLock`和它的`Condition`来实现线程安全。  

非阻塞队列则是利用`CAS`方法实现线程安全。  


## 如何选择合适自己的阻塞队列？  

先来看下常见线程池选择了哪些阻塞队列：   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/threadpool_and_blocking_queue.png)  


因此，我们可以从下面`三`个角度来分析自己的需求：  

第一，功能  

任务是否需要`排序`？`延迟执行`？  


第二， 容量和扩容  

容量是否需要固定？还是需要扩容？  

比如：`ArrayBlockingQueue`的容量固定，且无法扩容，  

而`LinkedBlockingQueue`容量为`Integer.MAX_VALUE`。  


第三， 性能  

比如：`LinkedBlockingQueue`由于持有两把锁，所以它的操作`粒度更细`，因此并发性能更好，  

同时，`SynchronousQueue`性能往往优于其他实现，因为它只需要`一直传递`，而不需要`存储`。  


