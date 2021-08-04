---
title: LinkedList源码分析
katex: false
tags: java
categories: java
abbrlink: 43075
date: 2021-08-04 17:59:43
---
```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>,Deque<E>,Cloneable,java.io.Serializable
```
- LinkedList是一个继承于AbstractSequentialList的双向链表
- LinkedList可以被当作堆栈、队列或双端队列进行操作
- LinkedList实现List接口，能对它进行队列操作
- LinkedList实现Deque接口，能将其当作双端队列使用
- LinkedList实现CLoneable接口，即覆盖clone函数，能克隆
- LinkedList实现java.io.Serializable接口，支持序列化
- LinkedList非同步的，不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法；`List list= Collections.synchronizedList(new LinkedList());`

[![47ba0f0d8b574a4460b5574e78352032.md.jpg](http://whh.plus:7007/images/2021/08/04/47ba0f0d8b574a4460b5574e78352032.md.jpg)](http://whh.plus:7007/image/r4k)

<!-- more -->
**为什么要继承自AbstractSequentialList**
AbstractSequentialList 实现了get(int index)、set(int index, E element)、add(int index, E element) 和 remove(int index)这些骨干性函数,降低了List接口的复杂度。这些接口都是随机访问List，通过AbstractSequentialList自己实现一个列表，只需要扩展此类，并提供 listIterator() 和 size() 方法的实现即可。若要实现不可修改的列表，则需要实现列表迭代器的 hasNext、next、hasPrevious、previous 和 index 方法。

![LinkedList类图关系](http://whh.plus:7007/images/2021/08/04/050053434697439.png)

## LinkedList数据结构
![数据结构](http://whh.plus:7007/images/2021/08/04/LinkedListE58685E983A8E7BB93E69E84.jpg)
双向链表节点定义
```java
private static class Node<E> {
    E item; //节点值
    Node<E> next; //后继节点
    Node<E> prev; //前驱节点

    Node(Node<E> prev, E element, Node<E> next){
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## LinkedList源码分析

### 构造方法

**空构造方法**
```java
public LinkedList(){

}
```
**用已有的集合创建链表的构造方法**
```java
public LinkedList(Collection<? extends E>c) {
    this();
    addAll(c);
}
```

### add()方法
add(E e)方法：将元素添加到链表尾部
```java
public boolean add(E e){
    linkLast(e); //就调用这一个方法
    return true;
}

//链接e作为最后一个元素
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l,e,null);
    last = newNode; //新建节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;   //后继指针指向新建的节点
    size++;
    modCount++;
}

//链接在开头
private void linkFirst(E e){
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null,e,f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

//链接在中间
void linkBefore(E e,Node<E> succ){
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred,e,succ);
    succ.prev = newNode;
    if(pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```
**add(int index,E e)**:在指定位置添加元素
```java
public void add(int index,E element) {
    checkPositionIndex(index);  //检查index是否处在[0,size]之间

    if(index == size)   //添加在链表尾部
        linkLast(element);
    else
        linkBefore(element,node(index));
}
```
**addAll(Collection c)**：将集合插入到链表尾部
```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size,c);
}
```
**addAll(int index,Collection c)**：将集合从指定位置开始插入
```java
public boolean addAll(int index, Collection<? extends E> c) {
    // 1. 检查index[0,size]
    checkPositionIndex(index);

    // 2. toArray()方法把集合的数据存到对象数组中
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;
    
    // 3. 得到插入位置的前驱节点和后继节点
    Node<E> pred,succ;
    // 如果插入位置为尾部，前驱节点为last，后继节点为null
    if(index ==size) {
        pred = last;
        succ =null;
    }else {
        //调用node得到前驱后续
        succ = node(index);
        pred = succ.prev;
    }

    // 4. 遍历数据将数据插入
    for(Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        //创建新节点
        Node<E> newNode = new Node<>(pred,e,null);
        //如果插入位置在链表头部
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    // 如果插入位置在尾部，重置last节点
    if (succ == null){
        last = pred;
    }else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```
四个步骤：
1. 检查index是否在size之内
2. 将集合的数据存入对象数据中toArray()
3. 得到插入位置的前驱节点和后继节点
4. 遍历数据，将数据插入到指定位置

**addFirst(E e)**：将元素添加到链表头部
```java
public void addFirst(E e){
    linkFirst(e);
}
```
**addLast(E e)**：将元素添加到链表尾部，与add(E e)一样
```java
public void addLast(E e){
    linkLast(e);
}
```

### get(int index) 根据位置取数据的方法
```java
public E get(int index) {
    //检查index在[0,size]之间
    checkElementIndex(index);
    //调用Node(index)去找到index对应的node并返回它的值
    return Node(index).item;
}
```

**获取头节点(index=0)数据方法**
```java

```