---
title: 算法学习
katex: false
tags: 算法
categories: 算法
abbrlink: 52784
date: 2021-01-16 16:08:05
---

## 一、数据结构的存储方式

数据结构的存储方式只有两种：数组(顺序存储)和链表(链式存储)

**队列、栈**这两种数据结构既可以使用链表也可以使用数组实现。用数组实现，就要处理扩容缩容问题；用链表实现，需要更多的内存空间存储节点指针

**图**的两种表达方式，邻接表就是链表，邻接矩阵是二维数组。邻接矩阵判断连通性迅速，并可以进行矩阵运算解决一些问题，但如果图比较稀疏很耗费空间；邻接表比较节省空间，但很多操作的效率比不过邻接矩阵

**散列表**通过散列函数把键映射到一个大数组里。解决散列冲突的方法，拉链法需要链表特性，操作简单，但需要额外的空间存储指针；线性探查法需要数组特性，以便连续寻址，不需要指针的存储空间，但操作稍微复杂些

**树**，用数组实现就是**堆**，**堆**是完全二叉树，用数组存储不需要节点指针，操作比较简单；用链表实现就是很常见的树，因为不一定是完全二叉树，所以不适合用数组存储。在链表**树**结构之上，又衍生出各种巧妙的设计，比如二叉搜索树、AVL树、红黑树、区间树、B树等等

**数组**:紧凑连续存储，可以随机访问，通过索引快速找到对应元素，而且相对节约存储空间，但内存空间必须一次性分配够，如果需要扩容，则需重新分配一块更大的空间，再把数据全部复制过去，时间复杂度为O(N)；如果进行插入和删除操作，每次必须搬移后面的所有数据以保持连续，时间复杂度O(N)

**链表**：元素不连续，靠指针指向下一个元素的位置，不存在扩容问题，如果知道某一元素的前驱和后驱，操作指针即可删除或插入元素，时间复杂度O(1)。因为存储空间不连续，无法根据索引算出对应元素地址，不能随机访问；由于每个元素必须存储前后元素位置的指针，会消耗更多的存储空间

## 二、数据结构的基本操作

对于任何数据结构，其基本操作无非遍历+访问，再具体一点就是：增删改查

各种数据结构的遍历+访问两种形式：线性的和非线性的

线性就是`for/while`迭代为代表，非线性就是递归为代表

数组遍历框架，典型的线性迭代结构：

```java
void traverse(int[] arr){
    for (int i=0;i<arr.length;i++){
        // 迭代访问 arr[i]
    }
}
```

链表遍历框架，兼具迭代和递归结构：

```java
/*基本的单链表节点*/
class ListNode{
    int val;
    ListNode next;
}

void traverse(ListNode head){
    for (ListNode p = head;p != null;p=p.next){
        // 迭代访问p.val
    }
}

void traverse(ListNode head){
    // 递归访问 head.val
    traverse(head.next);
}
```

二叉树遍历框架，典型的非线性递归遍历结构：

```java
/*基本的二叉树节点*/
class TreeNode{
    int val;
    TreeNode left,right;
}
void traverse(TreeNode root){
    traverse(root.left);
    traverse(root.right);
}
```

N叉树遍历框架：

```java
/*基本的N叉树节点*/
class TreeNode{
    int val;
    TreeNode[] children;
}
void traverse(TreeNode root){
    for (TreeNode child: root.children)
        traverse(child);
}
```

N叉树的遍历又可以扩展为图的遍历，图就是好几个N叉树的结合体。若图出现环，则用布尔数组visited做标记

## 算法刷题指南

**先刷二叉树**

**因为二叉树是最容易培养框架思维的，而且大部分算法技巧，本质上都是树的遍历问题。**

```java
void traverse(TreeNode root){
    //前序遍历
    traverse(root.left)
    //中序遍历
    traverse(root.right)
    //后序遍历
}
```

感谢**labuladong**

参考:
[https://labuladong.gitbook.io/algo/](https://labuladong.gitbook.io/algo/)