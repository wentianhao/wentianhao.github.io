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
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E element(){
    return getFirst();
}

public E peek() {
    final Node<E> f = first;
    return (f==null) ? null : f.item;
}

public E peekFirst() {
    final Node<E> f = first;
    return (f==null) ? null : f.item;
}
```
getFirst(),element(),peek(),peekFirst()这四个获取头节点方法的区别在于对链表为空时的处理，是抛出异常还是返回null,其中getFirst()和element()方法将会在链表为空是抛出异常

element()方法的内部就是使用getFirst()实现，在链表为空时，抛出NoSuchElementException异常

**获取尾节点(index=-1)数据方法**
```java
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```
getLast()方法在链表为空时，抛出异常，peekLast()方法不会，只是返回null

### 根据对象得到索引的方法
int indexOf(Object o)：从头遍历找
```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null){
        // 从头遍历
        for(Node<E> x = first;x != null;x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    }else {
        //从头遍历
        for (Node<E> x = first;x!=null;x = x.next){
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```
**int lastIndexOf(Object o)**：从尾遍历找
```java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        //从尾遍历
        for(Node<E> x = last;x!=null;x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    }else {
        for(Node<E> x = last;x != null;x = x.prev){
            index--;
            if(o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### 检查链表是否包含某个对象的方法
**contains(Object o)**：检查对象o是否存在链表中
```java
public boolean contains(Object o){
    return indexOf(o) != -1;
}
```

### 删除方法
**remove(),removeFirst(),pop()**：删除头节点
```java
public E pop() {
    return removeFirst();
}

public E remove() {
    return removeFirst();
}

public E removeFirst() {
    final Node<E> f = first;
    if(f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

private E unlinkFirst(Node<E> f) {
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```
**removeLast(),pollLast()**：删除尾节点
```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

private E unlinkLast(Node<E> l) {
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null;
    last = prev;
    if(prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```
removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

**remove(Object o)**：删除指定元素
```java
public boolean remove(Object o) {
    //如果删除对象为null
    if (o == null){
        //从头遍历
        for(Node<E> x = first;x != null; x = x.next) {
            //找到元素
            if (x.item == null){
                //从链表中移除找到的元素
                unlink(x);
                return true;
            }
        }
    }else {
        for(Node<E> x=first; x != null; x = x.next){
            if(o.equals(x.item)){
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    //删除前驱节点
    if (prev == null)
        first = next; //如果删除的是头节点，令头节点指向后继节点
    else {
        prev.next = next;
        x.prev = null;
    }
    //删除后继节点
    if (next == null)
        last = prev;
    else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
```
当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

**remove(int index)**：删除指定位置的元素
```java
public E remove(int index) {
    //检查index
    checkElementIndex(index);
    //删除节点
    return unlink(node(index));
}
```

## LinkedList 类常用方法测试
```java
package list;

import java.util.Iterator;
import java.util.LinkedList;

public class LinkedListDemo {
    public static void main(String[] srgs) {
        //创建存放int类型的linkedList
        LinkedList<Integer> linkedList = new LinkedList<>();
        /************************** linkedList的基本操作 ************************/
        linkedList.addFirst(0); // 添加元素到列表开头
        linkedList.add(1); // 在列表结尾添加元素
        linkedList.add(2, 2); // 在指定位置添加元素
        linkedList.addLast(3); // 添加元素到列表结尾
        
        System.out.println("LinkedList（直接输出的）: " + linkedList);

        System.out.println("getFirst()获得第一个元素: " + linkedList.getFirst()); // 返回此列表的第一个元素
        System.out.println("getLast()获得第最后一个元素: " + linkedList.getLast()); // 返回此列表的最后一个元素
        System.out.println("removeFirst()删除第一个元素并返回: " + linkedList.removeFirst()); // 移除并返回此列表的第一个元素
        System.out.println("removeLast()删除最后一个元素并返回: " + linkedList.removeLast()); // 移除并返回此列表的最后一个元素
        System.out.println("After remove:" + linkedList);
        System.out.println("contains()方法判断列表是否包含1这个元素:" + linkedList.contains(1)); // 判断此列表包含指定元素，如果是，则返回true
        System.out.println("该linkedList的大小 : " + linkedList.size()); // 返回此列表的元素个数

        /************************** 位置访问操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.set(1, 3); // 将此列表中指定位置的元素替换为指定的元素
        System.out.println("After set(1, 3):" + linkedList);
        System.out.println("get(1)获得指定位置（这里为1）的元素: " + linkedList.get(1)); // 返回此列表中指定位置处的元素

        /************************** Search操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.add(3);
        System.out.println("indexOf(3): " + linkedList.indexOf(3)); // 返回此列表中首次出现的指定元素的索引
        System.out.println("lastIndexOf(3): " + linkedList.lastIndexOf(3));// 返回此列表中最后出现的指定元素的索引

        /************************** Queue操作 ************************/
        System.out.println("-----------------------------------------");
        System.out.println("peek(): " + linkedList.peek()); // 获取但不移除此列表的头
        System.out.println("element(): " + linkedList.element()); // 获取但不移除此列表的头
        linkedList.poll(); // 获取并移除此列表的头
        System.out.println("After poll():" + linkedList);
        linkedList.remove();
        System.out.println("After remove():" + linkedList); // 获取并移除此列表的头
        linkedList.offer(4);
        System.out.println("After offer(4):" + linkedList); // 将指定元素添加到此列表的末尾

        /************************** Deque操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.offerFirst(2); // 在此列表的开头插入指定的元素
        System.out.println("After offerFirst(2):" + linkedList);
        linkedList.offerLast(5); // 在此列表末尾插入指定的元素
        System.out.println("After offerLast(5):" + linkedList);
        System.out.println("peekFirst(): " + linkedList.peekFirst()); // 获取但不移除此列表的第一个元素
        System.out.println("peekLast(): " + linkedList.peekLast()); // 获取但不移除此列表的第一个元素
        linkedList.pollFirst(); // 获取并移除此列表的第一个元素
        System.out.println("After pollFirst():" + linkedList);
        linkedList.pollLast(); // 获取并移除此列表的最后一个元素
        System.out.println("After pollLast():" + linkedList);
        linkedList.push(2); // 将元素推入此列表所表示的堆栈（插入到列表的头）
        System.out.println("After push(2):" + linkedList);
        linkedList.pop(); // 从此列表所表示的堆栈处弹出一个元素（获取并移除列表第一个元素）
        System.out.println("After pop():" + linkedList);
        linkedList.add(3);
        linkedList.removeFirstOccurrence(3); // 从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表）
        System.out.println("After removeFirstOccurrence(3):" + linkedList);
        linkedList.removeLastOccurrence(3); // 从此列表中移除最后一次出现的指定元素（从尾部到头部遍历列表）
        System.out.println("After removeFirstOccurrence(3):" + linkedList);

        /************************** 遍历操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.clear();
        for (int i = 0; i < 100000; i++) {
            linkedList.add(i);
        }
        // 迭代器遍历
        long start = System.currentTimeMillis();
        Iterator<Integer> iterator = linkedList.iterator();
        while (iterator.hasNext()) {
            iterator.next();
        }
        long end = System.currentTimeMillis();
        System.out.println("Iterator：" + (end - start) + " ms");

        // 顺序遍历(随机遍历)
        start = System.currentTimeMillis();
        for (int i = 0; i < linkedList.size(); i++) {
            linkedList.get(i);
        }
        end = System.currentTimeMillis();
        System.out.println("for：" + (end - start) + " ms");

        // 另一种for循环遍历
        start = System.currentTimeMillis();
        for (Integer i : linkedList)
            ;
        end = System.currentTimeMillis();
        System.out.println("for2：" + (end - start) + " ms");

        // 通过pollFirst()或pollLast()来遍历LinkedList
        LinkedList<Integer> temp1 = new LinkedList<>();
        temp1.addAll(linkedList);
        start = System.currentTimeMillis();
        while (temp1.size() != 0) {
            temp1.pollFirst();
        }
        end = System.currentTimeMillis();
        System.out.println("pollFirst()或pollLast()：" + (end - start) + " ms");

        // 通过removeFirst()或removeLast()来遍历LinkedList
        LinkedList<Integer> temp2 = new LinkedList<>();
        temp2.addAll(linkedList);
        start = System.currentTimeMillis();
        while (temp2.size() != 0) {
            temp2.removeFirst();
        }
        end = System.currentTimeMillis();
        System.out.println("removeFirst()或removeLast()：" + (end - start) + " ms");
    }
}
```

## 参考
[JavaGuide-LinkedList](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)
[Java集合---LinkedList源码解析](https://www.cnblogs.com/ITtangtang/p/3948610.html)