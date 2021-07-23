---
title: java基础
katex: true
tags: java
categories: java
date: 2021-07-20 21:19:46
---

Java基础概念与常识
<!-- more -->
## Java基础语法

**Java语言有哪些特点?**

1. 简单易学
2. 面向对象(封装、继承、多态)
3. 支持网络编程并且很方便
4. 支持多线程
5. 平台无关性
6. 可靠性
7. 安全性
8. 编译与解释并存

**JVM vs JDK vs JRE**

**JVM**

java虚拟机(JVM)是运行Java字节码的虚拟机。JVM有针对不同系统的特定实现，目的是使用相同的字节码，输出相同的结果。

**什么是字节码? 采用字节码的好处？**

在Java中,JVM可以理解的代码就叫做字节码(扩展名为.class的文件)，它不面向任何特定的处理器，只面向虚拟机。Java语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释性语言可移植的特点。所以Java程序运行时比较高效，而且由于字节码并不针对一种特定的机器，因此Java程序无须重新编译便可在多种不同操作系统的计算机上运行

Java程序从源代码到运行一般有下面3步：

.java文件(源代码)---JDK中的javac编译--->.class文件(JVM可理解的字节码)---JVM--->机器可执行的二进制机器码

JVM类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对较慢，而且有些方法和代码块是经常需要被调用的(热点代码)，所以后面引进了JIT编译器，JIT编译器属于运行时编译，当JIT编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次直接使用。机器码的运行效率是高于Java解释器的，这也解释了Java是编译与解释并存的语言

> HotSpot采用了惰性评估的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码(热点代码)，这也就是JIT所需编译的部分。JVM会根据代码每次被执行的情况收集信息并相应的做出一些优化，因此执行的次数越多，速度越快。

> JDK 9 引入了一种新的编译模式AOT(Ahead of time Compilation),直接将字节码编译成机器码，避免了JIT预热等各方面的开销。JDK支持分层编译和AOT协作使用，但是AOT编译器的编译质量不如JIT编译器

**总结**

Java虚拟机(JVM)是运行Java字节码的虚拟机，JVM有针对不同系统的特定实现，目的是使用相同的字节码，都会给出相同的结果。字节码和不同系统的JVM实现是Java语言“一次编译，随时可运行”的关键所在。

**JDK和JRE**

JDK是Java development kit缩写，是功能齐全的Java SDK。拥有JRE的一切，还有编译器javac和工具jdb和javadoc。能够创建和编译程序

JRE是Java运行的环境，它是运行已编译Java程序所需的所有内容的集合，包括jvm、java类库、Java命令和其他一些基础构建，但是不能用来创建新程序

**为什么Java语言“编译与解释并存？”**

高级编程预压按照程序的执行方式分为编译型和解释性两种。

编译型语言：编译器针对特定的操作系统将源代码一次性翻译成可被该平台执行的机器码；

解释性语言：解释器对源程序逐行解释成特定平台执行的机器码并立即执行

Java语言既具有编译型语言的特征，也具有解释性语言的特征。因为Java语言要经过先编译，后解释两个步骤，由Java编写的程序首先经过编译步骤，生成字节码(.class文件)；这种字节码必须由Java解释器来解释执行。因此可认为Java语言编译与解释并存。


**Java和C++的区别？**
- 都是面向对象的语言，都支持封装、继承 、多态
- Java不提供指针来直接访问内存，程序内存更安全
- Java的类是单继承，c++支持多继承；Java的类不可以多继承，接口可以多继承
- Java有自动内存垃圾回收机制(GC)，不需要手动释放无用内存
- c++同时支持方法重载和操作符重载，Java只支持方法重载

## 基本语法

**字符型常量和字符串常量的区别？**
1. 形式：字符常量是单引号引起的一个字符，字符串常量是双引号引起的0个或多个字符
2. 含义：字符常量相当于ASCII值，可以参加表达式运算；字符串常量代表一个地址值（该字符串在内存中存放位置）
3. 占内存大小：字符常量只占2个字节；字符串常量占若干个字节

| 基本类型 | 大小 | 最小值 | 最大值|包装器类型
|:--: |:--:| :---:| :---:|:---:|
boolean | -|-|-|Boolean
byte|8bits|-128| 127| Byte
short|16bots| -2^15|2^15-1 | Short
char|16bits| -2^15|2^15-1 | Character
int | 32bits| -2^31|2^31-1|Integer
float|32bits|-2^31|2^31-1|Float
long|64bits| -2^63| 2^63-1| Long
double| 64bits|-2^63|2^63-1|Double
void |-|-|-|Void

**Java泛型**
Java泛型(generics)是JDK 5中引入的一个新特性，泛型提供编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，所操作的数据类型被指定为一个参数

Java的泛型是伪泛型，因为Java在编译期间，所有的泛型信息都会被擦掉，也就是通常所说的类型擦除

```java
List<Integer> list = new ArrayList<>();
list.add(12);
// 这里直接添加会报错
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add",Object.class);
// 但是通过反射添加，是可以的
add.invoke(list,"k1");

System.out.println(list)
```

1. 泛型类
```java
/**
此处T 可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
在实例化泛型类时，必须指定T的具体类型
*/
public class Generic<T>{
    private T key;

    public Generic(T key){
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}

```

实例化泛型类
```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

2. 泛型接口
```java
public interface Generator<T>{
    public T method();
}
```
实现泛型接口，不指定类型
```java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method(){
        return null;
    }
}
```
实现泛型接口，指定类型
```java
class GeneratorImpl implements Generator<String>{
    @Override
    public String method(){
        return "hello";
    }
}
```
3. 泛型方法
```java
public static <E> void printArray(E[] inputArray){
    for(E element:inputArray){
        System.out.println("%s,element);
    }
    System.out.println();
}
```
使用
```java
// 创建不同类型数组:Integer 、Double 、Character
Integer[] intArray = {1,2,3,4};
String[] stringArray = {"hello","world"};
printArray(intArray);
printArray(stringArray);
```
常用的通配符：T、E、K、V、?
- ？：表示不确定的Java类型
- T(type)：表示具体的一个Java类型
- K V(key,value)分别代表Java键值中的Key Value
- E(element)代表Element

**==和equals的区别**
对于基本数据类型来说，==比较的是值，对于引用数据类型来说，==比较的是对象内存地址

> java只有值传递，对于==来说，不管是比较基础数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的是值还是对象的地址

equals（）作用不能用于判断基本数据类型的变量，只能用来判断两个对象是否相等。equals（）方法存在于Object类中，而Object类是所有类的直接或间接父类

Object类equals()方法：
```java
public boolean equals(Object obj){
    return (this == obj);
}
```

equals()方法存在两种使用情况：
- 类没有覆盖equals()方法：通过equals()比较该类的两个对象时，等价于通过“==”比较两个对象，使用的默认是Object类equals()方法
- 类覆盖了equals()方法：一般我们都覆盖equals()方法来比较两个对象中的属性是否相等；若属性相等，则返回true
```java
public class test{
    public static void main(String[]args){
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab");  // b为另一个引用，对象的内容一样
        String aa = "ab"; //放在常量池中
        String bb = "ab"; //从常量池查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a==b)   //false,非同一对象
            System.out.println("a==b");
        if (a.equals(b))    // true
            System.out.println("aEQb");
        if (42 == 42.0) //true
            System.out.println("true");
    }
}
```
