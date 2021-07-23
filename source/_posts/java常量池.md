---
title: java常量池
katex: false
tags: java
categories: java
abbrlink: 31451
date: 2021-07-22 10:39:46
---

对常量池理解的补充说明
<!-- more -->

## 常量池分类
常量池大体可以分为：静态常量池、运行时常量池
- 静态常量池：存在*.class文件中，class文件中的常量池不仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间。这种常量池主要用于存放两大类常量：字面量(Literal)和符号引用量(Symbolic References)。
    - 字面量：Java语言层面常量的概念，由字母，数字构成的字符串或数值，只能作为右值出现，例如文本字符串、声明为final的常量值等；
    - 符号引用量：1.类和接口的全限定名 2. 字段名称和描述符  3. 方法名称和描述符
- 运行时常量池：jvm虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存在方法区中

通常说的常量池指的是运行时常量池，所以讨论也是运行时常量池

## 字符串常量池
```java
String a  = "abc";
String b = new String("abc");
System.out.println(a==b);   // false
```
a作为字面量一开始存储在class文件中，之后运行期，转存方法区中；b 对象存储在堆中。

实例
```java
String s1 = "hello";
String s2 = "hello";
String s3 = "hel" + "lo";
String s4 = "hel" + new String("lo");
String s5 = new String("hello");
String s6 = s5.intern();
String s7 = "h";
String s8 = "ello";
String s9 = s7 + s8;
String s10 = new String("hello");

System.out.println(s1==s2);   // true
System.out.println(s1==s3);   // true
System.out.println(s1==s4);   // false
System.out.println(s1==s6);   // true
System.out.println(s1==s9);   // false
System.out.println(s4==s5);   // false
System.out.println(s10==s5);   // false
```
分析：
1. s1 == s2 都指向了方法区常量池中的 "hello"
2. s1 == s3 做+运算的时候，会进行优化，自动生成“hello”赋值给s3，因此也是true
3. s1 == s4 s4分别用了常量池中的字符串和存放对象的堆中的字符串，做+运算的时候会进行动态调用，最后生成的仍是一个String对象存放在堆中
4. s1 == s6 intern()方法首先在常量池中查找是否存在一份equal相等的字符串，如果有的话返回该字符串的引用，没有的话就将它加入到字符串常量池中，所以存在于class常量池并非固定不变，可以用intern()方法加入新的
5. s1 == s9 在Java9中，使用的是动态调用，返回的是一个新的String对象。s4、s5、s9、s10均不是指向同一块内存

## 需要注意的特例
1. 常量拼接
```java
public static final String a = "123";
public static final String b = "456";
public static void main(String[]args){
    String c = "123456";
    String d = a + b;
    System.out.println(c==d);   // true
}
```
final 类型的常量存在静态常量池中，做+运算的时候，会进行优化，自动生成“123456”赋值给d

2.static 静态代码块
```java
public static final String a;
public static final String b;

static{
    a = "123";
    b = "456";
}

public static void main(String[]args){
    String c = "123456";
    String d = a+b;
    System.out.println(c==d);   // false
}
```
上个例子在编译期间，就已经确定a和b，但是在这段代码中，编译器static不执行，a和b的值未知，static代码块，在初始化时被执行，初始化属于类加载的一部分，属于运行期，动态返回String类型对象，存在堆中，c在方法区常量池中

## 包装类的常量池技术(缓存)
自动装箱：valueOf() 

```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```


自动拆箱：intValue()




