---
layout: post
title: 二叉搜索树 - 前序遍历
category: algorithm
tags: [algorithm]
excerpt: Pre-order Traversal
---

## `Pre-order traversal using stack and loop`  

使用`栈`和`循环`实现`前序遍历`时，需要注意：先插入当前节点的右子树，然后才是左子树喔~  


``` java
@Setter
@Getter
public class TreeNode {
    public int val;
    private TreeNode leftNode;
    private TreeNode rightNode;
    private TreeNode parentNode;

    public TreeNode(int val){
        this.val  = val;
        leftNode  = null;
        rightNode = null;
    }
}

/**
 * Pre-order traversal using stack and while loop.
 */
public static void preOrderTraversalUsingStackAndLoop(TreeNode node){
    if (node == null) return;

    Stack<TreeNode> stack = new Stack<>();
    stack.push(node);
    System.out.println("PreOrder traversal using stack and loop: ");

    while (!stack.isEmpty()){

        TreeNode curNode = stack.pop();
        System.out.print(curNode.getVal() + " ");
        if (curNode.getRightNode() != null){
            stack.push(curNode.getRightNode());
        }
        if (curNode.getLeftNode() != null){
            stack.push(curNode.getLeftNode());
        }
    }
}

@Test
public void preOrderTraversalUsingStackAndLoopTest(){
    TreeUtils tu = new TreeUtils();
    tu.addTreeNode(4);
    tu.addTreeNode(2);
    tu.addTreeNode(6);
    tu.addTreeNode(1);
    tu.addTreeNode(3);
    tu.addTreeNode(5);
    tu.addTreeNode(7);
    BinaryTreePrinter.printNode(tu.root);

    TreeUtils.preOrderTraversalUsingStackAndLoop(tu.root);
}
```

结果如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/pre_order_traversal_using_stack_and_loop.png)

<br>
## `Pre-order traversal using recursive`  

使用`递归`方式实现`前序遍历`时，需要注意：当前节点为`null`时，应直接跳过。  

``` java
 /**
 * Pre-order traversal using recursion.
 */
public static void preOrderTraversalUsingRecursion(TreeNode node){
    if (node == null) return;

    System.out.print(node.getVal() + " ");
    preOrderTraversalUsingRecursion(node.getLeftNode());
    preOrderTraversalUsingRecursion(node.getRightNode());
}

@Test
public void preOrderTraversalRecursiveTest(){
    TreeUtils tu = new TreeUtils();
    tu.addTreeNode(4);
    tu.addTreeNode(2);
    tu.addTreeNode(6);
    tu.addTreeNode(1);
    tu.addTreeNode(3);
    tu.addTreeNode(5);
    tu.addTreeNode(7);
    BinaryTreePrinter.printNode(tu.root);

    System.out.println("PreOrder traversal using recursion: ");
    TreeUtils.preOrderTraversalUsingRecursion(tu.root);
}
```

详细执行流程与结果如下所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/pre_order_traversal_recursively.png)

<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/pre_order_traversal_using_recursion.png)
