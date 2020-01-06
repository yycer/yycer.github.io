---
layout: post
title: 二叉搜索树 - 基础
category: algorithm
tags: [algorithm]
excerpt: The Basic of Binary Search Tree
---

## 如何打印一棵二叉搜索树？  

``` java
public class BinaryTreePrinter {

    public static void printNode(TreeNode root) {
        int maxLevel = BinaryTreePrinter.maxLevel(root);
        /**
         * singletonList(T o): return ann immutable list containing only the specified object.
         */
        printNodeInternal(Collections.singletonList(root), 1, maxLevel);
    }

    private static void printNodeInternal(List<TreeNode> treeNodes, int level, int maxLevel) {
        if (treeNodes.isEmpty() || BinaryTreePrinter.isAllElementsNull(treeNodes))
            return;

        int floor = maxLevel - level;
        int endgeLines = (int) Math.pow(2, (Math.max(floor - 1, 0)));
        int firstSpaces = (int) Math.pow(2, (floor)) - 1;
        int betweenSpaces = (int) Math.pow(2, (floor + 1)) - 1;

        BinaryTreePrinter.printWhitespaces(firstSpaces);

        List<TreeNode> newTreeNodes = new ArrayList<>();
        for (TreeNode treeNode : treeNodes) {
            if (treeNode != null) {
                System.out.print(treeNode.getVal());
                newTreeNodes.add(treeNode.getLeftNode());
                newTreeNodes.add(treeNode.getRightNode());
            } else {
                newTreeNodes.add(null);
                newTreeNodes.add(null);
                System.out.print(" ");
            }

            BinaryTreePrinter.printWhitespaces(betweenSpaces);
        }
        System.out.println("");

        for (int i = 1; i <= endgeLines; i++) {
            for (int j = 0; j < treeNodes.size(); j++) {
                BinaryTreePrinter.printWhitespaces(firstSpaces - i);
                if (treeNodes.get(j) == null) {
                    BinaryTreePrinter.printWhitespaces(endgeLines + endgeLines + i + 1);
                    continue;
                }

                if (treeNodes.get(j).getLeftNode() != null)
                    System.out.print("/");
                else
                    BinaryTreePrinter.printWhitespaces(1);

                BinaryTreePrinter.printWhitespaces(i + i - 1);

                if (treeNodes.get(j).getRightNode() != null)
                    System.out.print("\\");
                else
                    BinaryTreePrinter.printWhitespaces(1);

                BinaryTreePrinter.printWhitespaces(endgeLines + endgeLines - i);
            }

            System.out.println("");
        }

        printNodeInternal(newTreeNodes, level + 1, maxLevel);
    }

    private static void printWhitespaces(int count) {
        for (int i = 0; i < count; i++)
            System.out.print(" ");
    }

    private static int maxLevel(TreeNode treeNode) {
        if (treeNode == null)
            return 0;

        return Math.max(BinaryTreePrinter.maxLevel(treeNode.getLeftNode()),
                BinaryTreePrinter.maxLevel(treeNode.getRightNode())) + 1;
    }

    private static boolean isAllElementsNull(List list) {
        for (Object object : list) {
            if (object != null)
                return false;
        }

        return true;
    }
}
```

<br>
## 如何创建一棵二叉搜索树？  

什么是二叉搜索树？来看下`wiki`给出的定义：  

> 二叉搜索树，又称二叉查找树、有序二叉树，是指一棵空树或者具有下列性质的二叉树：  
若任意节点的左子树非空，则左子树上所有节点的值均小于其根节点的值；  
若任意节点的右子树非空，则右子树上所有节点的值均大于其根节点的值；  
任意节点的左、右子树也分别为二叉查找树；  
且没有键值相等的节点。  

举个例子，我们想要在如下二叉搜索树中添加一个值为`3`的节点：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/build_tree.png)  


``` java
public class TreeUtils {

    public TreeNode root;

    public void addTreeNode(int val){
        root = addTreeNodeRecursively(root, val);
    }

    private TreeNode addTreeNodeRecursively(TreeNode curNode, int val) {
        if (curNode == null){
            return new TreeNode(val);
        }
        if (val < curNode.getVal()){
            curNode.setLeftNode(addTreeNodeRecursively(curNode.getLeftNode(), val));
        }
        else if (val > curNode.getVal()){
            curNode.setRightNode(addTreeNodeRecursively(curNode.getRightNode(), val));
        }
        // Binary search tree has no nodes with same value.
        else {
            return curNode;
        }
        return curNode;
    }
}


@Test
public void addAndPrintTreeNodeTest(){
    TreeUtils tu = new TreeUtils();
    tu.addTreeNode(4);
    tu.addTreeNode(2);
    tu.addTreeNode(6);
    tu.addTreeNode(3);
    BinaryTreePrinter.printNode(tu.root);
}
```

结果如下:  
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/print_simple_binary_search_tree.png)  


<br>
## 判断一棵二叉搜索树是否包含某个节点  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/tree_contain_node.png)  


``` java
/**
 * Determine the tree contains a tree node.
 */
public static boolean containTreeNode(TreeNode root, int val){
    if (val < 0 || root == null) return false;
    if (val == root.getVal())    return true;

    return val < root.getVal() ?
           containTreeNode(root.getLeftNode(), val) :
           containTreeNode(root.getRightNode(), val);
}

@Test
public void containTreeNodeTest(){
    TreeUtils tu = new TreeUtils();
    tu.addTreeNode(4);
    tu.addTreeNode(2);
    tu.addTreeNode(6);
    tu.addTreeNode(3);
    BinaryTreePrinter.printNode(tu.root);

    // true
    boolean contain4 = TreeUtils.containTreeNode(tu.root, 3);

    // false
    boolean contain2 = TreeUtils.containTreeNode(tu.root, 1);
}
```