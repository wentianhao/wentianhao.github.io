---
title: HashMap源码解读
katex: true
tags: java
categories: java
date: 2021-07-21 20:00:0
---

阿里面试：详细描述一下HashMap中put方法的整个过程

京东面试：HashMap扩容的标准和扩容的过程怎么实现

## HashMap

HashMap主要用来存放键值对，基于哈希表的Map接口实现实现，是常用的Java集合之一，是非线程安全的。

HashMap可以存储null的key和value,但null作为键只能有一个，null作为值可以有多个

JDK1.8之前HashMap由数组+链表组成，数组是HashMap主体，链表则是主要为了解决哈希冲突而存在的(拉链法解决冲突).

JDK1.8之后HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值(默认为8)(将链表转换成红黑树前会判断，如果当前数组的长度小于64,那么会选择先进行扩容，而不是转换为红黑树)时，将链表转化为红黑树，减少搜索时间

HashMap默认的初始化大小为16.之后每次扩容，容量变为原来的2倍，HashMap总是使用2的幂作为哈希表的大小
<!-- more -->

## 数据结构分析

### jdk1.8之前

JDK1.8之前HashMap底层是数组+链表结合在一起使用，也就是链表散列。

HashMap通过key的hashCode经过扰动函数处理过后得到hash值，然后通过`(n-1) & hash`

### jdk1.8

数组 + 链表 + 红黑树 JDK8

```java
static class Node<K,V> implements Map.Entry<K,V>{
    final int hash; //数据存放的位置与它有关
    final K key;
    V value;

}
```

put()方法：

1. 判断数组是否为空
2. 数组为空，则初始化数组
3. 数组不为空，则确定数据位置
4. 判断该位置是否为空
5. 如果位置为空，则直接存放该数据
6. 如果位置不为空，判断key是否相同
7. key相同，则进行值的覆盖，返回旧的值
8. 如果key不相同，链表找next属性是否为null(找的过程中会出现什么问题？) 

key相同（进行2次key判断？）、

链表太长导致查询效率下降
满足 >= 8 >= 64  转换为红黑树
不满足 则扩容


9.  

