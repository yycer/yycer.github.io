---
layout: post
title: Java - [容器]Queue
category: collection
tags: [spring, java]
excerpt: Java Collection Queue
---

`Update 2020_0425`

本文主要介绍下`Java`集合中的队列，整体关系图如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/queue_framework.png)  

再来看下`Queue`接口中的六个方法：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/queue_methods.png)  

接下来从四个维度分析三种队列：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/collection_four_dimension.png)  

## `PriorityQueue`  

`PriorityQueue`根据`元素的大小`进行重新排序，而不是维护了`元素插入的原始顺序`。   

``` java
@Test
void priorityQueueTest(){
    PriorityQueue<Integer> pq = new PriorityQueue<>();
    /** offer() and add() **/
    pq.offer(5);
    pq.offer(2);
    pq.offer(3);
    pq.offer(1);
    pq.offer(1);
    // pq.offer(null);

    /** peek() and element() **/
    System.out.println("Just peek: " + pq.peek());

    /** poll() and remove() **/
    System.out.println(pq);
    while (!pq.isEmpty()){
        System.out.println("Do   poll: " + pq.poll());
    }
}

// Just peek: 1
// [1, 1, 3, 5, 2]
// Do   poll: 1
// Do   poll: 1
// Do   poll: 2
// Do   poll: 3
// Do   poll: 5
```
可以得出以下结论：  

- 是否允许添加`null`？ `否`  
- 是否允许添加重复元素？ `是`  
- 是否维持插入顺序？ `否` 
- 是否线程安全？ `否`  

## `ArrayDeque`  


`ArrayDeque`代表一个`双端队列`，可同时实现`队列`和`栈`。    

下面来看下三者对应的方法：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/queue_stack_deque.png)  


再来看下具体代码实现：  


``` java
@Test
void arrayDequeTest(){
    /** As stack **/
    System.out.println("/** As stack **/");
    ArrayDeque<Integer> adStack = new ArrayDeque<>();
    adStack.push(5); // addFirst()
    adStack.push(2);
    adStack.push(3);
    adStack.push(3);
    // adStack.push(null);
    System.out.println(adStack);

    while (!adStack.isEmpty()){
        System.out.println("Polled element in adStack is " + adStack.poll()); // pollFirst()
    }

    System.out.println("/** As queue **/");
    /** As queue **/

    ArrayDeque<Integer> adQueue = new ArrayDeque<>();
    adQueue.offer(5); // offerLast()
    adQueue.offer(2);
    adQueue.offer(3);
    System.out.println(adQueue);
    while (!adQueue.isEmpty()){
        System.out.println("Polled element in adQueue is " + adQueue.poll()); // pollFirst();
    }
}
// /** As stack **/
// [3, 3, 2, 5]
// Polled element in adStack is 3
// Polled element in adStack is 3
// Polled element in adStack is 2
// Polled element in adStack is 5
// ------------------------------
// /** As queue **/
// [5, 2, 3]
// Polled element in adQueue is 5
// Polled element in adQueue is 2
// Polled element in adQueue is 3
```


可以得出以下结论：  

- 是否允许添加`null`？ `否`  
- 是否允许添加重复元素？ `是`  
- 是否维持插入顺序？ `否，栈[LIFO: Last In First Out]、队列[FIFO: First In First Out]`  
- 是否线程安全？ `否`  


## `LinkedList`  

这个类很厉害，同时实现了`List`和`Deque`接口。  

有个问题： `ArrayQueue`和`LinkedList`的区别是什么？  

其实看名称就能猜出一二， 

前者内部是用的是一个`Object[]`数组保存集合中的元素，因此`随机访问`元素时性能更好； 

而后者用链表保存集合中的元素，所以`插入`、`删除`元素时性能更佳。  

``` java
@Test
void linkedListTest(){
    LinkedList<Integer> ll = new LinkedList<>();
    ll.add(5);
    ll.add(2);
    ll.add(3);
    ll.add(3);
    ll.add(null);
    System.out.println(ll);
    // [5, 2, 3, 3, null]
}
```

- 是否允许添加`null`？ `是`  
- 是否允许添加重复元素？ `是`  
- 是否维持插入顺序？ `是` 
- 是否线程安全？ `否`  


总结：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/queue_features.png)  

## `Reference`  
- `《疯狂Java讲义》 - 8.5 Queue集合`  
