---
layout: post
title: 算法与数据结构 - 链表总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - LinkedList
---


本文来总结下链表的核心概念，以及归纳下相关题目。  



## 核心概念  

- 链表本质  

> 通过`指针`将一组`零散`的内存块`串联`起来。  


- 有哪几种常见的链表？  

`单向链表、双向链表、环状链表`  


- 哨兵的作用？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linked_list_dummyNode.png)  

图中的`dummyNode`，也称之为哨兵，作用就是简化头尾节点的操作。  


- 避免指针丢失和内存泄漏  

举个例子：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linked_list_insert_origin.png)  

如何在`1`和`3`之间插入`2`？  

一开始你的思路可能是这样的： 

``` java
curNode.next = newNode;
newNode.next = curNode.next;
```

我们来分析一下：  

当执行第一行代码时，`curNode: 1 -> 2`，注意此时`1`与`3`节点间的关系已断开！  

因此，当执行到第二行代码时，`newNode`指向的就是自己。  

那么如何调整？  

更换两行代码顺序，先将新节点`2`指向当前节点的下一个节点`3`，然后令当前节点`1`指向新节点`2`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linked_list_insert.png)  

- 考虑边界  

可以从以下四个角度考虑：  

- 单链表  
- 单节点链表  
- 双节点链表    
- 头尾节点  


## 实战题目  

- *1、[[206] Reverse Linked List](http://yaoyichen.cn/algorithm/2020/03/24/leetcode-206.html){:target="_blank"}  
- 2、[[141] Linked List Cycle](http://yaoyichen.cn/algorithm/2020/03/26/leetcode-141.html){:target="_blank"}  
- *3、[[21] Merge Two Sorted Lists](http://yaoyichen.cn/algorithm/2020/03/25/leetcode-21.html){:target="_blank"}  
- *4、[[19] Remove Nth Node From End of List](http://yaoyichen.cn/algorithm/2020/05/03/leetcode-19.html){:target="_blank"}  
- 5、[[83] Remove Duplicates from Sorted List](http://yaoyichen.cn/algorithm/2020/03/25/leetcode-83.html){:target="_blank"}  
- 6、[[203] Remove Linked List Elements](http://yaoyichen.cn/algorithm/2020/05/03/leetcode-203.html){:target="_blank"}  
- 7、[[876] Middle of the Linked List](http://yaoyichen.cn/algorithm/2020/03/24/leetcode-876.html){:target="_blank"}  
- 8、[[160] Intersection of Two Linked Lists](http://yaoyichen.cn/algorithm/2020/03/26/leetcode-160.html){:target="_blank"}  
- 9、[[2] Add Two Numbers](http://yaoyichen.cn/algorithm/2020/05/03/leetcode-2.html){:target="_blank"}  
- *10、[[234] Palindrome Linked List](http://yaoyichen.cn/algorithm/2020/05/03/leetcode-234.html){:target="_blank"}  
- *11、[[142] Linked List Cycle II](http://yaoyichen.cn/algorithm/2020/07/03/leetcode-142.html){:target="_blank"}  
- *12、[[92] Reverse Linked List II](http://yaoyichen.cn/algorithm/2020/07/11/leetcode-92.html){:target="_blank"}  
- *13、[[82] Remove Duplicates from Sorted List II](http://yaoyichen.cn/algorithm/2020/07/11/leetcode-82.html){:target="_blank"}  
- *14、[[143] Reorder List](http://yaoyichen.cn/algorithm/2020/07/11/leetcode-143.html){:target="_blank"}  
