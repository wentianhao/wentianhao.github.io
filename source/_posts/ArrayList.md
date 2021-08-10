---
title: ArrayList源码分析
katex: false
tags: java
categories: java
abbrlink: 33908
date: 2021-08-03 16:52:58
---

感觉ArrayList源码分析还是要单独开一章博客，在容器里写的话有点冗余
<!-- more -->
## ArrayList源码分析
主要对增删改查的源代码、扩容规则、ArrayListIterator、ArrayList的三种遍历进行分析
```java
//默认初始容量大小
private static final int DEFAULT_CAPACITY = 10;
// 空数组，用于空实例
private static final Object[] EMPTY_ELEMENTDATA = {};
// 用于默认大小空实例的共享空数组实例
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
// 保存ArrayList数据的数组
transient Object[] elementData;
//ArrayList包含元素个数
private int size;
```
### 构造函数
两个空数组区别：第一次添加元素时知道该 elementData 从空的构造函数还是有参构造函数被初始化的。以便确认如何扩容。

当进行ArrayList()无参构造时，构造一个容量大小为10的空的list集合，但构造函数只是给elementData赋值了一个空的数组，当第一次执行add新增元素操作时容量扩大为10
```java
// 默认无参构造函数
// DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10
public ArrayList(){
  this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
**构造一个初始容量为initialCapacity的ArrayList**
```java
// 带初始容量参数的构造函数(用户可以在创建ArrayList对象时指定集合的初始大小)
public ArrayList(int initialCapacity){
  if(initialCapacity > 0){
    // 如果传入的参数大于0，创建initialCapacity大小的数组
    this.elementData = new Object[initialCapacity];
  }else if(initialCapacity == 0){
    //如果传入的参数为0，创建空数组
    this.elementData = EMPTY_ELEMENTDATA;
  }else{
    //否则抛出异常
    throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
  }
}
```
小结：
- 当使用无参构造函数时，是将DEFAULTCAPACITY_EMPTY_ELEMENTDATA赋值给elementData。
- 当initialCapacity = 0时，将EMPTY_ELEMENTDATA 赋值给elementData
- 当initialCapacity > 0时，初始化了一个大小为initialCapacity的object数组并赋值给elementData

**使用指定Collection来构造ArrayList的构造函数**
```java
//构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序
public ArrayList(Collection<? extends E> c) {
  //将指定集合转换为数据
  elementData = c.toArray();
  // 如果elementData数组的长度不为 0
  if((size= elementData.length) != 0) {
    // 如果elementData不是Object类型数据 (c.toArray可能返回的不是Object类型的数组，所以下面进行判断)
    if(elemnetData.getClass() != Object[].class)
      // 将原本不是Object类型的elementData数组的内容，赋值给Object类型的elementData数组
      elementData = Arrays.copeOf(elementData,size,Object[].class);
  } else{
    //其他情况，空数组代替
    this.elementData = EMPTY_ELEMENTDATA;
  }
}
```
将Collection转化为数组并赋值给elementData，把elementData中的元素的个数赋值为size。如果size不为0，则判断elementData的class类型是否为Object[],不是则做一次转换。如果size为0，则使用EMPTY_ELEMENTDATA赋值给elementData,相当于new ArrayList(0).

### 扩容机制
```java
// ArrayList的扩容机制提高了性能，如果每次只能扩充一个，那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这点。减少增量重新分配的次数
public void ensureCapacity(int minCapacity){
  //如果是无参构造，miXExpand为0，否则为10
  int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) ? 0:DEFAULT_CAPACITY;
  // 如果最小容量大于已有的最大容量，判断是狗需要扩容
  if (minCapacity > minExpand){
    ensureExplicitCapacity(minCapacity);
  }
}

//得到最小容量
private void ensureExplicitCapacity(int minCapacity){
  modCount++;
  
  if(minCapacity - elementData.length > 0 )
    grow(minCapacity);
}

//要分配的最大数组大小
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

// ArrayList扩容核心方法
private void grow(int minCapacity){
  int oldCapacity = elementData.length;
  //将oldCapacity 右移一位，其效果相当于oldCapacity /2，位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  //检查新容量是否小于最小需要容量，若新容量小于则将最小需要容量赋值给新容量
  if(newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
  //再检查新容量是否超出ArrayList定义的最大容量，超出则调用hugeCapacity()来比较minCapacity和MAX_ARRAY_SIZE
  if(newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
  elementData = Arrays.copyOf(elementData,newCapacity);
}

//比较minCapacity 和 MAX_ARRAY_SIZE.如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
private hugeCapacity(int minCapacity){
  if(minCapacity < 0)
    throw new OutOfMemoryError();
  return (minCapacity > MAX_ARRAY_SIZE) ? INTEGER.MAX_VALUE : MAX_ARRAY_SIZE;
}

```

### System.arraycopy()和Arrays.copyOf()方法
ArrayList中大量调用了这俩方法。看看这俩方法的区别

#### System.arraycopy()

```java
//native方法，复制数组
/** src:源数组
*** srcPos:源数组中的起始位置
*** dest: 目标数组
*** destPos: 目标数组中的起始位置
*** length：要复制的数组的元素的数量
*/
public static native void arraycopy(Object src, int srcPos,Object dest, int destPos,int length);
```
测试
```java
public class ArraycopyTest {
    int []a = new int[10];
    a[0] = 0;
    a[1] = 1;
    a[2] = 2;
    a[3] = 3;
    a[4] = 4;
    a[5] = 5;
    System.arraycopy(a,2,a,3,3);
    // 0,1,2,2,3,4,0,0,0,0
    for (int i : a){
        System.out.print(i + " ");
    }
    System.out.println("--------------------");
    a[2] = 20;
    // 0,1,20,2,3,4,0,0,0,0
    for (int i : a){
        System.out.print(i + " ");
    }
}
```
将 源数组从源数组起始位置开始的length个元素赋值到目标数组从目标位置开始的length个元素。

#### Arrays.copyOf()方法
```java
public static int[] copeOf(int[] original, int newLength){
    //申请一个新的数组
    int[] copy = new int[newLength];
    //调用System.arraycopy,将源数组中数据拷贝，返回新数组
    System.arraycopy(original,0,copy,0,Math.min(original.length,newLength));
    return copy;
}
```
Arrays.copyOf()方法主要是为了给原有数组扩容

System.arraycopy()需要目标数组，将源数组拷贝到目标数组里，而且可以选择拷贝到起点和长度，以及放入目标数组的位置。

Arrays.copyOf()则是系统自动在内部新建一个数组，返回该数组

### 增 add()
ArrayList添加元素的操作，涉及2个方法`add(E e)`和`add(int index,E e)`

**add(E e)**：直接在尾部添加一个元素
```java
public boolean add(E e){
  ensureCapacityInternal(size+1);
  elementData[size++] = e;
  return true;
}
```

**ensureCapacityInternal(int minCapacity)**：确认集合容量大小
```java
private void ensureCapacityInternal(int minCapacity) {
	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
		minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
	}
	ensureExplicitCapacity(minCapacity);  //判断是否需要扩容，调用grow(int minCapacity)函数
}
```
判断如果是默认无参构造的，在第一次添加元素时初始化容量为10。

**add(int index,E element) && addAll()**
```java
public void add(int index,E element){
    //add和addAll使用的rangeCheck的一个版本,判断index是否在[0,size]之间
    rangeCheckForAdd(index);
    ensureCapacityInternal(size+1);
    System.arraycopy(elementData,index,elementData,index+1,size-index);
    elementData[index] = element;
    size++;
}
```
add(int index,E element)先调用rangeCheckForAdd(int index)方法对index进行界限检查，然后调用ensureCapacityInternal方法保证capacity足够大，再将从index开始之后的成员后移一个变量，将element插入index位置，最后size+1

```java
//按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
public boolean addAll(Collection<? extends E> c){
    Object []a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);
    System.arraycopy(a,0,elementData,size,numNew);
    size += numNew;
    return numNew != 0;
}
```

```java
//将指定集合中的所有元素插入到此列表中，从指定的位置开始
public boolean addAll(int index, Collection<? extends E> c){
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacity(size + numNew);

    int numMoved = size - index;
    if(numMoved > 0)
        System.arraycopy(elementData,index,elementData,index + numNew,numMoved);
    Systen.arraycopy(a,0,elementData,index,numNew);
    size + = numNew;
    return numNew != 0;
}
```
add(int index, E element)，addAll(Collection<? extends E> c)，addAll(int index, Collection<? extends E> c) 操作是都是先对集合容量检查 ，以确保不会数组越界。然后通过 System.arraycopy() 方法将旧数组元素拷贝至一个新的数组中去。

### 删 remove()
删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。
```java
public E remove(int index){
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if(numMoved > 0)
        System.arraycopy(elementData,index+1,elementData,index,numMoved);
    elementData[--size] = null;
    return oldValue;
}
```
从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。如果此列表包含指定的元素,返回true
```java
public boolean remove(Object o){
    if(o == null){
        for(int index =0;index< size;index++){
            if(elementData[index] == null){
                fastRemove(index);
                return true;
            }
        }
    } else {
        for(int index =0;index<size;index++)
            if(o.equals(elementData[index])){
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

```java
private void fastRemove(int index){
    modCount++;
    int numMoved = size - index - 1;
    if(numMoved > 0)
        System.arraycopy(elementData,index+1,elementData,index,numMoved);
    elementData[--size] = null;
}
```
当调用 remove(int index) 时，首先会检查 index 是否合法，然后再判断要删除的元素是否位于数组的最后一个位置。如果 index 不是最后一个，就再次调用 System.arraycopy() 方法拷贝数组。说白了就是将从 index + 1 开始向后所有的元素都向前挪一个位置。然后将数组的最后一个位置空，size - 1。如果 index 是最后一个元素那么就直接将数组的最后一个位置空，size - 1即可。 当我们调用 remove(Object o) 时，会把 o 分为是否为空来分别处理。然后对数组做遍历，找到第一个与 o 对应的下标 index，然后调用 fastRemove 方法，删除下标为 index 的元素。其实仔细观察 fastRemove(int index) 方法和 remove(int index) 方法基本全部相同。

clear():从列表中删除所有元素。
```java
public void clear(){
    modCount++;
    for(int i=0;i<size;i++)
        elementData[i] = null;
    size = 0;
}
```
### 改 set()
用指定的元素替换此列表中指定位置的元素。
```java
public E set(int index,E element){
    // 对index界限检查
    rangeCheck(index);

    E oldValue  = elementData(index);
    elementData[index] = element;
    //返回原来的元素
    return oldValue;
}
```

### 查 get()
```java
public E get(int index){
    rangeCheck(index);
    return elementData(index);
}
```
由于 ArrayList 底层是基于数组实现的，所以获取元素就相当简单了，直接调用数组随机访问即可。

### 迭代器 iterator
在集合中，用for循环遍历集合时不能使用remove操作进行删除，因为remove操作会改变集合的大小，从而造成数组下标越界，更严重则抛出异常。
```java
public static void main(String[]args){
    List<String> list = new ArrayList<>();
    list.add("a");
    list.add("b");
    list.add("c");
    list.add("d");
    list.add("e");
    // 错误的删除方法
    for(String s : list){   //报错 java.util.ConcurrentModificationException
        if(s.equals("c"))
            list.remove(s);
    }

    // 正确的删除方法
    Iterator<String> iterator = list.iterator();
    while(iterator.hasNext()){
        String s = iterator.next();
        if (s.equals("c")){
            iterator.remove();
        }
    }
}
```
iterator方法返回一个Itr对象
```java
public Iterator<E> iterator(){
    return new Itr();
}
```

```java
private class Itr implements Iterator<E>{
    int cursor; //代表下一个要访问的元素下标
    int lastRet = -1;  //代表上一个要访问的元素下表
    int expectedModCount = modCount;  //代表对ArrayList修改次数的期望值，初始为modCount
    
    // 如果下一个等于集合大小，则到最后了
    public boolean hasNext() {
		return cursor != size;
	}

    @SuppressWarnings("unchecked")
	public E next() {
        // 判断 expectedModCount 和 modCount 是否相等
		checkForComodification();
		int i = cursor;
        //对cursor进行判断，是否超过集合大小和数组长度
		if (i >= size)
			throw new NoSuchElementException();
		Object[] elementData = ArrayList.this.elementData;
		if (i >= elementData.length)
			throw new ConcurrentModificationException();
		cursor = i + 1;
        // 将cursor赋值给lastRet,并返回下标为lastRet的元素，cursor自增。
		return (E) elementData[lastRet = i];
	}

    //remove 先判断lastRet值
	public void remove() {
		if (lastRet < 0)
			throw new IllegalStateException();
		//检查expectedModCount 和 modCount 是否相等
        checkForComodification();
        //直接调用 ArrayList 的 remove 方法删除下标为 lastRet 的元素。然后将 lastRet 赋值给 cursor ，将 lastRet 重新赋值为 -1，并将 modCount 重新赋值给 expectedModCount。
		try {
			ArrayList.this.remove(lastRet);
			cursor = lastRet;
			lastRet = -1;
			expectedModCount = modCount;
		} catch (IndexOutOfBoundsException ex) {
			throw new ConcurrentModificationException();
		}
        //直接调用 ArrayList 的 remove 方法删除下标为 lastRet 的元素。然后将 lastRet 赋值给 cursor ，将 lastRet 重新赋值为 -1，并将 modCount 重新赋值给 expectedModCount。
	}

	final void checkForComodification() {
		if (modCount != expectedModCount)
			throw new ConcurrentModificationException();
	}
}
```
几点注意：
- 只能进行remove操作，Itr中没有add，clear
- 调用remove之前必须调用next。因为remove开始就对lastRet做校验。lastRet初始化为-1
- next之后只能调用一次remove。因为remove会将lastRet初始化为-1


**总结**
ArrayList的底层是基于数组实现容量大小动态可变。扩容机制首先扩容到原始容量1.5倍，如果1.5倍大小的话，将需要的容量赋值给newCapacity。如果1.5倍太大的话或者需要的容量太大，就直接newCapacity = (minCapacity > MAX_ARRAY_SIZE)?Integer.MAX_VALUE : MAX_ARRAY_SIZE;来扩容。扩容之后通过数组的拷贝来确保元素的准确性，所以尽量减少扩容操作。ArrayList的最大存储能力：Integer.MAX_VALUE。 **size**:集合存储的元素的个数。如果遍历remove，必须使用iterator。并且remove前需要next，next之后只能一次remove。

## **ArrayList源码注释**
```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;


public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

     //用于默认大小空实例的共享空数组实例。
      //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList 所包含的元素个数
     */
    private int size;

    /**
     * 带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //如果传入的参数大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果传入的参数等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //其他情况，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
        }
    }

    /**
     *默认无参构造函数
     *DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //将指定集合转换为数组
        elementData = c.toArray();
        //如果elementData数组的长度不为0
        if ((size = elementData.length) != 0) {
            // 如果elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组所以加上下面的语句用于判断）
            if (elementData.getClass() != Object[].class)
                //将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 其他情况，用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 修改这个ArrayList实例的容量是列表的当前大小。 应用程序可以使用此操作来最小化ArrayList实例的存储。
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
//那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        //如果是true，minExpand的值为0，如果是false,minExpand的值为10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //如果最小容量大于已有的最大容量
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取“默认的容量”和“传入参数”两者之间的最大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
  //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量，
        //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
        //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     *返回此列表中的元素数。
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回 true 。
     */
    public boolean isEmpty() {
        //注意=和==的区别
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true 。
     */
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。）
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）;
     *返回的数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返回其中。
     *否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。
     *如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null 。
     *（这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。）
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }

    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此列表中的指定位置插入指定的元素。
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
      //从列表中删除的元素
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
     *返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从列表中删除所有元素。
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除所有索引为fromIndex （含）和toIndex之间的元素。
     *将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合中包含的所有元素。
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。
     *返回的迭代器是fail-fast 。
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
```

## 参考
[ArrayList详解，看这篇就够了](https://blog.csdn.net/sihai12345/article/details/79382649)
[javaGuide-ArrayList](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ArrayList%E6%BA%90%E7%A0%81+%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6%E5%88%86%E6%9E%90)