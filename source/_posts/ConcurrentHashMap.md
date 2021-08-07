---
title: ConcurrentHashMap
katex: false
tags: java
categories: java
abbrlink: 58651
date: 2021-08-07 17:09:35
---
ConcurrentHashMap，线程安全的HashMap。
<!-- more -->
## ConcurrentHashMap 1.7
### 存储结构
![1.7](http://whh.plus:7007/images/2021/08/07/image-20200405151029416.png)
Java7中ConcurrentHashMap的存储结构如上图，ConcurrentHashMap由很多个segment组合，而每一个segment是一个类似于HashMap的结构，所以每一个HashMap的内部可以进行扩容。但是Segment的个数一旦初始化就不能改变，默认Segment个数是16个，默认支持最多16个线程并发
### 初始化
通过ConcurrentHashMap的无参构造探索ConcurrentHashMap的初始化流程
```java
// 创建一个默认的map 初始容量 16 ，负载因子 0.75 ，默认并发级别 16
// 默认初始化容量
static final int DEFAULT_INITIAL_CAPACITY = 16;
// 默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 默认并发级别
static final int DEFAULT_CONCURRENCY_LEVEL = 16;

public ConcurrentHashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

**有参构造函数内部实现逻辑**
```java
@SupperssWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,float loadFactor,int concurrencyLevel) {
    // 参数校验
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    // 校验并发级别大小，大于 1<<16，重置为 65536
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    int sshift = 0;
    int ssize = 1;
    // 这个循环可以找到 concurrencyLevel 之上最近的 2的次方值
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // 记录段偏移量
    this.segmentShift = 32 - sshift;
    // 记录段掩码
    this.segmentMask = ssize - 1;
    // 设置容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // c = 容量 / ssize ，默认 16 / 16 = 1，这里是计算每个 Segment 中的类似于 HashMap 的容量
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    //Segment 中的类似于 HashMap 的容量至少是2或者2的倍数
    while (cap < c)
        cap <<= 1;
    // 创建 Segment 数组，设置 segments[0]
    Segment<K,V> s0 = new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```
**总结 Java7 中 ConcurrentHashMap 初始化逻辑**
1. 必要参数检验
2. 检验并发级别大小，如果大于最大值，重置为最大值。无参构造默认值为16
3. 寻找并发级别concurrencyLevel之上最近的2的幂次方值，作为初始化容量大小，默认是16
4. 记录segmentShift偏移量，这个值为【容量 = 2的N次方】的N，在后面put计算位置时会用到。默认是 32-sshift = 28；
5. 记录segmentMask，默认 ssize - 1 = 16 -1 =15
6. 初始化segments[0]，默认大小为2，负载因子0.75，扩容阈值 2 * 0.75 = 1.5。插入第二个值时会进行扩容。

### put
```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // hash 值无符号右移 28位（初始化时获得），然后与 segmentMask=15 做与运算
    // 其实也就是把高4位与segmentMask（1111）做与运算
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // 如果查找到的 Segment 为空，初始化
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}

@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    // 判断 u 位置的 Segment 是否为null
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        // 获取0号 segment 里的 HashEntry<K,V> 初始化长度
        int cap = proto.table.length;
        // 获取0号 segment 里的 hash 表里的扩容负载因子，所有的 segment 的 loadFactor 是相同的
        float lf = proto.loadFactor;
        // 计算扩容阀值
        int threshold = (int)(cap * lf);
        // 创建一个 cap 容量的 HashEntry 数组
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) { // recheck
            // 再次检查 u 位置的 Segment 是否为null，因为这时可能有其他线程进行了操作
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 自旋检查 u 位置的 Segment 是否为null
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                // 使用CAS 赋值，只会成功一次
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```
1. 计算要put的key的位置，获得指定位置的Segment
2. 如果指定位置的Segment为空，初始化Segment
3. Segment.put插入key,value值

**初始化Segment流程**
1. 检查计算得到的位置的Segment是否为null
2. 为null，继续初始化，使用Segment[0]的容量和负载因子创建一个HashEntry数组。
3. 再次检查计算得到的指定位置的Segment是否为null
4. 使用创建的HashEntry数组初始化这个Segment
5. 自旋判断计算得到的指定位置的Segment是否为null,使用CAS在这个位置赋值为Segment.

**Segment.put插入流程**
```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 获取 ReentrantLock 独占锁，获取不到，scanAndLockForPut 获取。
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        // 计算要put数据的位置
        int index = (tab.length - 1) & hash;
        // CAS 获取 index 坐标的值
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first; ;) {
            if (e != null) {
                //检查是否key已经存在，如果存在，则遍历链表寻找位置，找到后替换value；
                K k;
                if ((k = e.key) ==|| (e.hash == hash && key.euqals(k))) {
                    oldValue = e.value;
                    if(!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            } else {
                // first 有值说明index位置已经有值，有冲突，链表头插法
                if(node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash,key,value,first);
                int c = count + 1;
                //容量大于扩容阈值，小于最大容量，进行扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    // index 位置赋值 node，node 可能是一个元素，也可能是一个链表的表头
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```
由于Segment继承了ReentrantLock,所以Segment内部可以很方便的获取锁
1. `tryLock()`获取锁，获取不到使用`scanAndLockForPut`方法继续获取
2. 计算put的数据要放入的index位置，然后获取这个位置上的HashEntry。
3. 遍历HashEntry,因为HashEntry可能是一个空元素，也可能是链表已存在
4. 如果这个位置上的HashEntry不存在，如果当前容量大于扩容阈值，小于最大容量，进行扩容；然后进行头插法插入
5. 如果这个位置上的HashEntry存在
6. 判断链表当前元素的key和hash值是否和要put的key和hash值相等，相等则替换值
7. 如果不一致，获取链表的下一个节点，直到发现相同进行值替换，或者链表没有相同，如果当前容量大于扩容阈值，小于最大容量，扩容；然后直接进行头插法插入。
8. 如果要插入的位置之前已经存在，替换后返回旧值，否则返回null

**scanAndLockForPut**：不断的自旋`tryLock()`获取锁。当自旋次数大于指定次数时，使用`lock()`阻塞获取锁。在自旋时顺表获取下hash位置的HashEntry。
```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) { 
    HashEntry<K,V>first = entryForHash(this,hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1;
    //自旋获取锁
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            // 自旋达到指定次数后，阻塞等到只到获取到锁
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

### 扩容rehash
ConcurrentHashMap的扩容只会扩容到原来的两倍。老数组里的数据移动到新数组时，要么位置不变，要么变为index + oldSize，参数里的node会在扩容之后使用链表头插法插入到指定位置
```java
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    // 老容量
    int oldCapacity = oldTable.length;
    // 新容量
    int newCapacity = oldCapacity << 1;
    // 新的扩容阈值
    threshold = (int)(newCapacity * loadFactor);
    // 创建新的数组
    HashEntry<K,V>[] newTable = (HashEntry<K,V> [])new HashEntry[newCapacity];
    // 新的掩码，默认2扩容后是4，-1后是3，二进制就是11。
    int sizeMask = newCapacity - 1;
    for(int i=0;i<oldCapacity;i++) {
        // 遍历老数组
        HashEntry<K,V>e = oldTable[i];
        if(e!=null){
            HashEntry<K,V> next = e.next;
            // 计算新的位置，新的位置只可能是不变或者是老位置+oldCapacity
            int idx = e.hash & sizeMask;
            if(next == null)
            // 如果当前位置还不是链表，只是一个元素，直接赋值
                newTable[idx] = e;
            else {
                // 如果是链表了
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                // 新的位置只可能是不变或者是老的位置+老的容量。
                // 遍历结束后，lastRun 后面的元素位置都是相同的
                for (HashEntry<K,V> last = next; last != null; last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // ，lastRun 后面的元素位置都是相同的，直接作为链表赋值到新位置。
                newTable[lastIdx] = lastRun;
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    // 遍历剩余元素，头插法到指定 k 位置。
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 头插法插入新的节点
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```
第一个for是为了寻找这样一个节点，这个节点后面的所有next节点的新位置都是相同的。然后把这个作为一个链表赋值到新位置。第二个for循环是为了把剩余的元素通过头插法插入到指定位置链表。