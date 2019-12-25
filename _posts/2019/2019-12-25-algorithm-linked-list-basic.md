---
layout: post
title: 链表基础
category: algorithm
tags: [algorithm]
excerpt: The Basic of Linked List
---

## 链表基础  

先来看下`wiki`对于`linked list`的定义：  


> In computer science, a linked list is a linear collection of data elements,  
> whose order is not given by their physical placement in memory. Instead, each element points to the next.  
> It is a data structure consisting of a collection of nodes which together represent a sequence.  
> In its most basic form, each node contains: data, and a reference (in other words, a link) to the next node in the sequence.  
> This structure allows for efficient insertion or removal of elements from any position in the sequence during iteration.

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/wiki-linked-list.png)  


``` java
@Setter
@Getter
public class Node {

    private String val;
    private Node   nextNode;

    public Node(String val, Node nextNode){
        this.val      = val;
        this.nextNode = nextNode;
    }
}
```

## 简单添加一个节点  

``` java
public class LinkedListUtils {

    public static Node addNode(Node node, String val){
        if (node == null){
            return new Node(val, null);
        }
        Node curNode = node;
        // 定位到最后一个节点。
        while (curNode.getNextNode() != null){
            curNode = curNode.getNextNode();
        }
        curNode.setNextNode(new Node(val, null));
        return node;
    }
}
```


## 打印链表  

``` java
public class LinkedListUtils {

    public static void printNode(Node node){
        String result;
        StringBuilder sb = new StringBuilder();

        while (node != null){
            sb.append(node.getVal());
            sb.append(" -> ");
            node = node.getNextNode();
        }
        String sbStr = new String(sb);
        int length = sbStr.length();
        if (length > 2){
            result = sbStr.substring(0, length - 4);
        } else {
            result = sbStr;
        }
        System.out.println(result);
    }
   
}

@SpringBootTest
public class LinkedListTest {

    public static Node node;

    static {
        // 1 -> 3 -> 5 -> 2 -> 7
        Node n1 = new Node("7", null);
        Node n2 = new Node("2", n1);
        Node n3 = new Node("5", n2);
        Node n4 = new Node("3", n3);
        node    = new Node("1", n4);
    }

    @Test
    public void addValTest(){
        // 1 -> 3 -> 5 -> 2 -> 7
        LinkedListUtils.printNode(node);
        Node node = LinkedListUtils.addNode(LinkedListTest.node, "10");
        
        // 1 -> 3 -> 5 -> 2 -> 7 -> 10
        LinkedListUtils.printNode(node);
    }
}
```

## 判断链表是否包含某个节点  


``` java
public class LinkedListUtils {
   
    public static boolean isContainNode(Node node, String val){
        while (node != null){
            if (val.equals(node.getVal())){
                return true;
            }
            node = node.getNextNode();
        }
        return false;
    }
}

@SpringBootTest
public class LinkedListTest {
    @Test
    public void isContainNode(){
        boolean isContainNode = LinkedListUtils.isContainNode(node, "1");
        // isContainNode: true
        System.out.println("isContainNode: " + isContainNode);
    }
}
```
