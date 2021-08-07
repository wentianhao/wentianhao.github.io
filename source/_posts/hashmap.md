---
title: HashMap源码解读
katex: false
tags: java
categories: java
abbrlink: 14882
date: 2021-07-21 20:00:00
---


阿里面试：详细描述一下HashMap中put方法的整个过程

京东面试：HashMap扩容的标准和扩容的过程怎么实现
<!-- more -->
## HashMap

HashMap主要用来存放键值对，基于哈希表的Map接口实现实现，是常用的Java集合之一，是非线程安全的。

HashMap可以存储null的key和value,但null作为键只能有一个，null作为值可以有多个

JDK1.8之前HashMap由数组+链表组成，数组是HashMap主体，链表则是主要为了解决哈希冲突而存在的(拉链法解决冲突).

JDK1.8之后HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值(默认为8)(将链表转换成红黑树前会判断，如果当前数组的长度小于64,那么会选择先进行扩容，而不是转换为红黑树)时，将链表转化为红黑树，减少搜索时间

HashMap默认的初始化大小为16.之后每次扩容，容量变为原来的2倍，HashMap总是使用2的幂作为哈希表的大小

## 数据结构分析

### jdk1.8之前

JDK1.8之前HashMap底层是数组+链表结合在一起使用，也就是链表散列。

HashMap通过key的hashCode经过扰动函数处理过后得到hash值，然后通过`(n-1) & hash`判断当前元素存放的位置(n指数组长度)，如果当前位置存在元素的话，判断该值与要存放元素的hash值以及key是否相同，如果相同直接覆盖，不相同通过拉链法解决冲突

扰动函数即HashMap的hash方法，使用扰动函数是为了减少碰撞

### JDK1.8 HashMap的hash方法
```java
static final int hash(Object key){
    int h;
    // key.hashCode();返回散列值hashCode
    // ^ ：按位异或
    // >>> ：无符号右移，忽略符号位，空位以0补齐
    // 高16位与低16位做异或操作
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // h = 0001000011111100 1000101011001001
//h>>>16 = 0000000000000000 0001000011111100
    // r = 0001000011111100 1001101000110101
}
```

### JDK1.7 HashMap的hash方法
```java
static int hash(int h){
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```
相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次

**拉链法**：将链表和数组结合，创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。
![拉链法](http://whh.plus:7007/images/2021/08/05/jdk1.8E4B98BE5898DE79A84E58685E983A8E7BB93E69E84.png)

### JDK1.8之后
JDK1.8之后在解决哈希冲突时有了较大的变化

当链表的长度大于 阈值(默认为8)时，首先调用`treeifyBin()`方法，这个方法会根据HashMap数组来决定是否转换为红黑树。只有当数组长度大于或等于64的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就只是执行`resize()`方法对数组进行扩容。
![tree](http://whh.plus:7007/images/2021/08/05/up-bba283228693dae74e78da1ef7a9a04c684.png)

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>,Cloneable,Serializable {
    //序列号
    private static final long serialVersionUID = 362498820763181265L;
    //默认的初始容量为 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于8时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶(bucket)上的结点数小于6时树就转成链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64
    // 存储元素的数组，总是2的幂次倍
    transient Node<K,V>[] table;
    // 存放具体元素的集
    transient Set<Map.entry<k,v>> entrySet;
     // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
}
```
- loadFactor 加载因子
loadFactor加载因子是控制数组存放数据的疏密程度，loadFactor越趋于1，数组中存放的数组越多，也就越密，也就会让链表的长度增加；loadFactor越小，也就越趋于0，数组中存放数据越少，也就越稀疏。

loadFactor太大导致查找元素效率低，太小导致数组利用率低，存放的数据会很分散，loadFactor默认0.75f是官方给的一个比较好的临界值

给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
- threshold
threshold = capacity * loadFactor，当 Size>=threshold的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准。**

**Node 节点类源码**
```java
static class Node<K,V> implements Map.Entry<K,V>{
    final int hash; //数据存放的位置与它有关,哈希值，存放元素到hashmap中时用来与其他元素hash值比较
    final K key;    // 键
    V value;    // 值
    // 指向下一个节点
    Node<K,V> next;
    Node(int hash,K key,V value,Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
     public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    // 重写hashCode()方法
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    //重写equals()方法
    public final boolean equals(Object o) {
        if(o == this)
            return true;
        if(o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key,e.getKey()) && Objects.equals(value,e.getValue()))
                return true;
        }
        return false;
    }
}
```
**树节点类源码**
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;   //父
    TreeNode<K,V> left; //左
    TreeNode<K,V> right;    //右
    TreeNode<K,V> prev; 
    boolean red;    //判断颜色
    TreeNode(int hash,K key,V val,Node<K,V> next) {
        super(hash,key,val,next);
    }
    //返回根节点
    final TreeNode<K,V> root() {
        for(TreeNode<K,V> r = this,p ; ; ){
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```
## HashMap源码分析

### 构造方法
HashMap有四种构造方法
```java
//默认构造方法
public HashMap(){
    this.loadFactor = DEFAULT_LOAD_FACTOR;
}

// 包含另一个Map的构造函数
public HashMap(Map<? extends K,? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m,false);//后面详细说明
}

// 指定容量大小的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//指定容量大小和加载因子的构造函数
public HashMap(int initialCapacity,float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity" + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```
**putMapEntries方法**
```java
final void putMapEntries(Map<? extends K,? extends V> m,boolean evict) {
    int s = m.size();
    if (s > 0){
        // 判断table是否已经初始化
        if (table == null) {
            //未初始化,s为m的实际元素个数
            float ft = ((float) s / loadFactor) + 1.0F;
            int t = ((ft < (float) MAXIMUM_CAPACITY) ? (int) ft : MAXIMUM_CAPACITY);
            //计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        } else if (s > threshold) {
            //已初始化，并且m元素个数大于阈值，进行扩容处理
            resize();
        }
        for(Map.Entry<? extends K,? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key),key,value,false,evict);
        }

    }
}
```

### put()方法：
HashMap只提供put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用。

**对putVal方法添加元素**
1. 如果定位到的数组位置没有元素，就直接插入
2. 如果定位到的数组位置有元素，就需要和要插入的key进行比较，如果key相同就直接覆盖，如果key不同，就判断p是否为一个树节点，如果是就调用`((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)
![put](http://whh.plus:7007/images/2021/08/05/putE696B9E6B395.png)

上图两个小问题
- 直接覆盖之后应该就会 return，不会有后续操作。参考 JDK8 HashMap.java 658 行
- 当链表长度大于阈值（默认为 8）并且 HashMap 数组长度超过 64 的时候才会执行链表转红黑树的操作，否则就只是对数组扩容。参考 HashMap 的 treeifyBin() 方法

```java
public V put(K key,V vlaue) {
    return putVal(hash(key),key,value,false,true);
}

final V putVal(int hash,K key,V value,boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab;
    Node<K,V> p;
    int n,i;
    // 步骤①：tab为空则创建
    // table未初始化或长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0) 
        n = (tab = resize()).length;
    // 步骤②：计算index，并对null做处理
    // (n-1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash,key,value,null);
    else {
        Node<K,V> e; 
        K k;
        //步骤③：节点key存在，直接覆盖value
        //比较桶中第一个元素(数组中的结点)的hash值相等，key 相等
        if (p.hash == hash && ((k = p.key) == key || (key != null) && key.equals(k)))
            // 将第一个元素赋值给e，用e来记录
            e = p;
        // 步骤④：判断该链为红黑树
        // hash值不相等，即key不相等，为红黑树结点
        // 如果当前元素类型为TreeNode，表示为红黑树，putValue返回待存放的node，e可能为null
        else if (p instanceof TreeNode)
            //放入树中
            e = (TreeNode<K,V> p).putTreeVal(this,tab,hash,key,value);
        // 步骤⑤：该链为链表
        // 链表结点
        else {
            //在链表最末插入结点
            for(int binCount = 0; ; ++ binCount){
                //到达链表底部
                if ((e = p.next) == null){
                    //在尾部插入新节点
                    p.next = newNode(hash,key,value,null);
                    // 结点数量达到阈值(8),执行treeifyBin方法
                    // 这个方法根据HashMap数组来决定是否转换为红黑树
                    // 只有数组长度大于等于 64 才会执行转换红黑树，以减少搜索时间，否则就只是对数组扩容
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab,hash);
                    break;
                }
                // 判断链表中的结点的key值与插入的元素的key是否相等
                if (e.hash == hash && ((k = e.key) == key || (k != null && key.equals(k))))
                    break;
                //用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在同种找到key值，hash值与插入元素相等的结点
        if (e != null){
            //记录e 的value
            V oldValue = e.value;
            // onluIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值代替
                e.value = value;
            //访问后回调
            afterNodeAcess(e);
            //返回旧值
            return oldValue;
        }
    }
    //结构性修改
    ++modCount;
    // 步骤⑥：超过最大容量就扩容
    //实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afrerNodeInsertion(evict);
    return null;
}
```
![put](http://whh.plus:7007/images/2021/08/06/20191214222552803.png)
①. 判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容
②. 根据键值对key计算hash值得到插入的数组索引i，如果table[i]桶为null，直接新建节点添加，转向⑥，如果table[i]桶不为空，转向③
③. 判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，相同指的是hashCode和equals均相同
④. 判断table[i]桶的数据类型是否为TreeNode,即该桶是否为红黑树，如果是红黑树，直接在树中插入键值对，否则转向⑤
⑤. 遍历table[i]，判断链表长度是否大于8，大于8的话判断数组是否大于等于64，否则就将链表转化为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作，遍历过程中，判断key若已经存在则直接覆盖
⑥. 插入成功后，判断实际存在的键值对数量size是否超过了最大容量threshold,如果超过了，进行扩容

put方法流程：

put方法调用了putVal方法，将计算出的hash值传入。在putVal方法中，先找到hash值对应的下标，如果数组中对应下标为空，直接将节点插入，如果不为空，分三种情况：
1. 如果要插入的节点的key正好与头节点的key相同，直接修改value值
2. 如果要插入的节点不在头节点，并且table中存储的为红黑树，在树中查找有无该key，如果存在，则直接修改value值，如果不存在，在树中插入该节点
3. 如果插入的节点不在头节点，并且table中存储的为链表，在链表中查找有无该key，如果存在，则直接修改value值，如果不存在，在链表的末尾插入该节点。
4. 最后完成插入操作，检查当前size是否超出最大容量，超出则resize()操作进行扩容。

**JDK1.7 put方法**
对于put方法的分析如下:
- 如果定位到的数组位置没有元素，就直接插入
- 如果定位到的数组位置有元素，遍历以这个元素为头节点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE)
        inflateTable(threshold);
    
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash,table.length);
    for(Entry<K,V> e = table[i];e!=null;e=e.next){
        Object k ;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recodeAcess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(hash,key,value,i); //再插入
    return null;
}
```

### remove方法
```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                            boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //当table不为空，并且hash对应的桶不为空时
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        //桶中的头节点就是我们要删除的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //用node记录要删除的头节点
            node = p;
        //头节点不是要删除的节点，并且头节点之后还有节点
        else if ((e = p.next) != null) {
            //头节点为树节点，则进入树查找要删除的节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            //头节点为链表节点
            else {
                //遍历链表
                do {
                    //hash值相等，并且key地址相等或者equals
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                            (key != null && key.equals(k)))) {
                            //node记录要删除的节点
                        node = e;
                        break;
                    }
                    //p保存当前遍历到的节点
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //我们要找的节点不为空
        if (node != null && (!matchValue || (v = node.value) == value ||
                                (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                //在树中删除节点
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            //我们要删除的是头节点
            else if (node == p)
                tab[index] = node.next;
            //不是头节点，将当前节点指向删除节点的下一个节点
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```
removeNode方法和putVal方法非常的像，这两者本来就是一个删一个增，所以在代码上有共性。

### get方法
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key),key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash,Object key) {
    Node<K,V>[] tab;
    Node<K,V> first,e;
    int n;
    K k;
    if ((tab = table)!=null && (n = tab.length) > 0 && (first = tab[(n-1) & hash]) != null){
        // 数组元素相等
        if (first.hash == hash && ((k = first.key) == key || key.equals(k)))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null){
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>) first).getTreeNode(hash,key);
            // 在链表中
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
            } while((e = e.next) != null);
        }
    }
    return null;
}
```
①. 通过hash & (n - 1) 获取该key对应的数据节点的数组位置
②. 判断首节点是否为空，为空直接返回空
③. 再判断首节点key是否和目标key相同，相同直接返回(首节点不区分链表还是红黑树)
④. 如果桶中不止一个节点，判断是树型节点还是链表，如果是树型节点，进入红黑树的取值流程，返回结果
⑤. 如果是链表，则遍历链表，判断与key相同的对象，返回结果。

### resize方法
resize方法就是对HashMap进行扩容，会伴随依次重新hash分配，并且遍历hash表中的所有元素，重新计算所有节点的hash值对应的小标，然后将节点转换到新table中，是非常耗时的。编程时，尽量避免resize。
```java
final Node<K,V>[] resize() {
    Node<K,V> [] oldTable = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap,newThr = 0;
    if (oldCap > 0) {
        //如果原容量已经达到最大容量了，无法进行扩容，直接返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            //阈值也变为原来的两倍
            newThr = oldThr << 1; // double threshold
    } 
     /**
        * 从构造方法我们可以知道
        * 如果没有指定initialCapacity, 则不会给threshold赋值, 该值被初始化为0
    	* 如果指定了initialCapacity, 该值被初始化成大于initialCapacity的最小的2的次幂
		* 这里这种情况指的是原table为空，并且在初始化的时候指定了容量，
		* 则用threshold作为table的实际大小
		*/
    else if (oldThr > 0)
        newCap = oldThr;
    //构造方法中没有指定容量，则使用默认值
    else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    /**从以上操作我们知道, 初始化HashMap时, 
    *  如果构造函数没有指定initialCapacity, 则table大小为16
    *  如果构造函数指定了initialCapacity, 则table大小为threshold,
    *  即大于指定initialCapacity的最小的2的整数次幂
    
    *  从下面开始, 初始化table或者扩容, 实际上都是通过新建一个table来完成
    */ 
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 这里table中存放的只是Node的引用，将oldTab[j]=null只是消除旧表的引用，但真正的node节点还在，只是现在由e指向它
                oldTab[j] = null;
                // 桶里只有一个节点，直接放入新桶
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 桶中为红黑树，则对树进行拆分
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 桶中为链表，对链表进行拆分    
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历该桶
                    do {
                        next = e.next;
                        // 原索引 找出拆分后仍处在同一个桶中的节点
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

链表的拆分

这里定义了四个变量：loHead，loTail,hiHead,hiTail两个头节点，两个尾节点

![链表](http://whh.plus:7007/images/2021/08/07/20190621145827456.png)

这张图中index=2的桶中有四个节点，在未扩容之前，它们的 hash& cap 都等于2。在扩容之后，它们之中2、18还在一起，10、26却换了一个桶。这就是这句代码的含义：选择出扩容后在同一个桶中的节点。
`if (e.hash & oldCap) == 0`

oldCap = 8, 8的二进制：1000
- 2的二进制：0010。 0010 & 1000 = 0000
- 10的二进制: 1010。 1010 & 1000 = 1000
- 18的二进制：10010。 10010 & 1000 = 0000
- 26的二进制：11010。 11010 & 1000 = 1000

从与操作后的结果可以看出来，2和18应该在同一个桶中，10和26应该在同一个桶中。

lo和hi这两链表的作用就是保存原链表拆分的两个链表。
```java
// 找到拆分后仍处于同一个桶的节点，将这些节点重新连接起来。
if ((e.hash & oldCap) == 0){
    //尾节点为空，说明lo链表是空的
    if (loTail == null)
        loHead = e;
    else 
        loTail.next = e;
    loTail = e;
} else {
    if (hiTail == null)
        hiTail = e;
    else
        hiTail.next = e;
    hiTail = e;
}
```

将拆分完的链表放进桶里的操作。只需将头节点放进桶里。newTab[j]和newTab[j + oldCap]分别代表了扩容之后原位置与新位置，就相当于之前那张图中的2和10.
```java
if (loTail != null){
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null){
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

**小结**
1. 什么时候进行resize操作？
有两种情况下会进行resize操作。一：初始化table；二：在size超过threshold之后进行扩容
2. 扩容的新数组容量为多大比较合适？
扩容后的数组应为原来数组的两倍，并且数组的大小必须是2的幂
3. 节点在转移的过程中是一个个节点复制还是一串一串的转移？
从源码中可以看出，扩容时是先找到拆分后处于同一个桶的节点，将这些节点连接好，然后把头节点存入桶中。
4. 为什么负载因子是0.75？
根据统计学的结果，hash冲突是符合泊松分布的，而冲突概率最小的是在7-8之间,当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为负载因子，每个碰撞位置的链表长度超过8个是几乎不可能的。
5. **HashMap如何扩容**
- 如果table==null,则HashMap初始化，生成空table返回
- 如果table不为空，需计算table的长度，newLength = oldLength << 1(如果oldLength已到上限，则newLength = oldLength)
- 遍历oldTable
- 首节点为空，循环结束
- 首节点不为空，无后续节点，重新计算hash位，本次循环结束
- 若有后续节点，当前是红黑树，走红黑树的重定位,红黑树是把构建新链表的过程变为构建两颗新的红黑树。定位都是使用`(e.hash & oldCap) == 0`判断
- 若当前是链表，通过 `(e.hash & oldCap) == 0`来判断是否需要移位，分为两类，一类在原位`hash`不动，一类移动到`hash + oldCap`位置

## **resize死循环**
**jdk7**对于HashMap节点重定位中，在resize()时会形成环形链表，然后导致get时死循环
![死锁](http://whh.plus:7007/images/2021/08/07/9b7636283ba0b8e17251e7a6d5d87635.png)

resize前的HashMap如下：
![resize前](http://whh.plus:7007/images/2021/08/07/20190128152737406.png)
这时，有两个线程需要插入到第四个节点，这个时候需要resize。线程二必须等线程一完成再resize。
![resize1](http://whh.plus:7007/images/2021/08/07/20190128152749145.png)
经过线程一resize后，发现a，b节点的顺序被反转了。这时候来看线程二
![resize2](http://whh.plus:7007/images/2021/08/07/20190128152809635.png)
1. 线程二开始只是获取a节点，还没获取他的next
2. 这时候线程一resize完成，`a.next = null,b.next = a;newTable[i] = b;`
3. 线程二开始执行，获取a节点，`a.next = null;`
4. 接着执行 `a.next = newTable[i];`这时候线程二就会形成`a->b->a`环形链表
5. 因为第三步a.next = null,因此c节点丢失了
6. 如果这时候来查找位于1节点的数据d，就会陷入死循环

**jdk8**resize是让节点的顺序发生改变，没有出现倒排问题。假设有两个线程，线程一执行完成，这时候线程二来执行
![jdk1.8](http://whh.plus:7007/images/2021/08/07/20190128152831296.png)
1. 因为顺序没变，所以node1.next还是node2,node2.next从node3变成了null
2. JDK8在遍历完所有节点之后，才对形成的两个链表进行关联table，不会像jdk7那样形成a-b-a环形链表问题
3. 但是如果并发了，Java的HashMap还是没有解决丢数据问题。但不会和jdk7有数据倒排以至于死循环问题。

HashMap设计时没有保证线程安全，在多线程时使用`ConcurrentHashMap`

## HashMap常用方法测试
```java
public class HashMapDemo {

    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<String, String>();
        // 键不能重复，值可以重复
        map.put("san", "张三");
        map.put("si", "李四");
        map.put("wu", "王五");
        map.put("wang", "老王");
        map.put("wang", "老王2");// 老王被覆盖
        map.put("lao", "老王");
        System.out.println("-------直接输出hashmap:-------");
        System.out.println(map);
        /**
         * 遍历HashMap
         */
        // 1.获取Map中的所有键
        System.out.println("-------foreach获取Map中所有的键:------");
        Set<String> keys = map.keySet();
        for (String key : keys) {
            System.out.print(key+"  ");
        }
        System.out.println();//换行
        // 2.获取Map中所有值
        System.out.println("-------foreach获取Map中所有的值:------");
        Collection<String> values = map.values();
        for (String value : values) {
            System.out.print(value+"  ");
        }
        System.out.println();//换行
        // 3.得到key的值的同时得到key所对应的值
        System.out.println("-------得到key的值的同时得到key所对应的值:-------");
        Set<String> keys2 = map.keySet();
        for (String key : keys2) {
            System.out.print(key + "：" + map.get(key)+"   ");

        }
        /**
         * 如果既要遍历key又要value，那么建议这种方式，因为如果先获取keySet然后再执行map.get(key)，map内部会执行两次遍历。
         * 一次是在获取keySet的时候，一次是在遍历所有key的时候。
         */
        // 当我调用put(key,value)方法的时候，首先会把key和value封装到
        // Entry这个静态内部类对象中，把Entry对象再添加到数组中，所以我们想获取
        // map中的所有键值对，我们只要获取数组中的所有Entry对象，接下来
        // 调用Entry对象中的getKey()和getValue()方法就能获取键值对了
        Set<java.util.Map.Entry<String, String>> entrys = map.entrySet();
        for (java.util.Map.Entry<String, String> entry : entrys) {
            System.out.println(entry.getKey() + "--" + entry.getValue());
        }

        /**
         * HashMap其他常用方法
         */
        System.out.println("after map.size()："+map.size());
        System.out.println("after map.isEmpty()："+map.isEmpty());
        System.out.println(map.remove("san"));
        System.out.println("after map.remove()："+map);
        System.out.println("after map.get(si)："+map.get("si"));
        System.out.println("after map.containsKey(si)："+map.containsKey("si"));
        System.out.println("after containsValue(李四)："+map.containsValue("李四"));
        System.out.println(map.replace("si", "李四2"));
        System.out.println("after map.replace(si, 李四2):"+map);
    }

}
```

## 参考
[HashMap的put方法的具体流程](https://www.cnblogs.com/fengyupinglan/p/14428758.html)
[HashMap之get方法详解](https://blog.csdn.net/weixin_39667787/article/details/86687414)
[深入理解HashMap（二）put方法解析](https://blog.csdn.net/weixin_41565013/article/details/93173607)
[深入理解HashMap（三）resize方法解析](https://blog.csdn.net/weixin_41565013/article/details/93190786)
[JavaGuide HashMap(JDK1.8)源码+底层数据结构分析](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90)
[HASHMAP负载因子为什么是0.75](https://www.cnblogs.com/theRhyme/p/10609207.html)
[HashMap之resize详解](https://blog.csdn.net/weixin_39667787/article/details/86678215)
[HashMap产生死循环死锁的原因图文极简说明](https://www.pianshen.com/article/62741427515/)