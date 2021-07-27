---
title: Java基础
katex: false
tags: java
categories: java
abbrlink: 23187
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
**说明**：
- String中的equals方法是被重写过的，因为Object的equals方法比较对象的内存地址，而String的equals方法比较的是对象的值
- 当创建String类型对象时，虚拟机会在常量池中查找也没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用，如果没有就在常量池中重新创建一个String对象。

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

**hashCode()与equals()**
重写equals()时必须重写hashCode()方法

1. hashCode()介绍
hashCode()作用是获得哈希码，也称散列码。作用是确定该对象在哈希表中的索引位置。hashCode()定义在 JDK 的 Object 类中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。另外需要注意的是： Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法通常用来将对象的 内存地址 转换为整数之后返回。
```java
public native int hashCode();
```
2. 为什么要有hashCode？
以HashSet检查重复为例；当把对象加入到HashSet时，首先对对象计算hashCode来判断对象加入的位置，然后与其他已经加入的对象的hashCode进行比较，如果相同的hashCode,则对象无重复；如果发现相同hashCode值，则会调用equals()方法来检查相同hashCode的对象是否真的相同。如果相同，则重复不加入对象操作，如果不同的话，就重新散列到其他位置。这样大大减少了equals()次数，提高执行效率。
3. 为什么重写equals()时必须重写hashCode()方法？
如果两个对象相等，那么hashCode一定相等，但是两个对象有相同的hashCode，却不一定是相等的。因此equals()方法被覆盖过，则hashCode()方法也必须被覆盖
4. 为什么两个对象有相同的hashCode，也不一定相等？
因为hashCode()所使用的哈希算法也许刚好会让多个对象传回相同的哈希值。

## 基本数据类型
**Java中几种基本数据类型、对应的包装类型、各占多少字节**
8种基本数据类型：6种数字类型:byte、short、int、long、float、double；1种字符类型:char;1种布尔类型:boolean

| 基本类型| 位数| 字节| 默认值|包装类
|:--:|:--:|:--:|:--:|:--:|
|byte| 8 | 1 | 0|Byte|
|short|16|2|0|Short|
|int | 32 | 4|0|Integer|
|long|64|8|0L|Long|
|float|32|4|0f|Float|
|double|64|8|0d|Double|
|char|16|2|'u0000'|Character|
|boolean|1||false|Boolean|

包装类型不赋值时为Null，基本数据类型直接存放在Java虚拟机栈中的局部变量表中，而包装类型属于对象类型，对象实例存在堆中。

## 自动装箱与拆箱
- 装箱：将基本类型用它们对应的引用各类型包装起来   valueof()
- 拆箱：将包装类型转换为基本数据类型            xxxValue()

```java
Integer i = 10; //装箱 < -- >  Integer i = Integer.valueOf(10)
int n = i; //拆箱      < -- >     int n = i.intValue()
```

## 8种基本类型的包装类喝常量池
Java基本类型的包装类的大部分都实现了常量池技术。Byte、Short、Integer、Long这四种包装类默认创建了数值[-128,127]的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean直接返回True/False

Integer缓存源码：
```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/

public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
      return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];
}
```
Character缓存源码:
```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {

    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}

```
Boolean缓存源码:
```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```
如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。

两种浮点数类型的包装类Float、Double没有实现常量池技术
```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1==i2)  //true

Float f1 = 333f;
Float f2 = 333f;
System.out.println(f1==f2)  //false

Double d1 = 1.2;
Double d2 = 1.2;
System.out.println(d1==d2) //false

Integer i3 = new Integer(33);
System.out.println(i1==i3)  //false
```

所有整型包装类对象之间值的比较，全部使用equals()方法比较

补充说明[java常量池](https://whh.plus/2021/07/22/java%E5%B8%B8%E9%87%8F%E6%B1%A0)

## 方法(函数)
在一个静态方法内调用一个非静态成员为什么是违法的？

结合JVM，静态方法属于类，在类加载的时候分配内存，可以通过类名直接访问。而非静态成员属于实例对象，只有在对象实例化之后才存在，然后通过类的实例对象去访问。在类的非静态成员不存在的时候静态成员已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作。

**静态方法和实例方法有什么不同？**
1. 在外部调用静态方法时，可以使用类名.方法名的方式，也可以使用对象名.方法名的方式。而实例方法只有后面这种方式，也就是说，调用静态方法可以无需创建对象。
2. 静态方法在访问本类的成员时，只允许访问静态成员(即静态成员和静态方法)，而不允许访问实例成员变量和实例方法；实例方法则无此限制

**为什么Java中只有值传递？**
按值调用：表示方法接收的是调用者提供的值；

按引用调用：表示方法接收的是调用者提供的变量地址

一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值

Java是按值调用，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。

**example**
```java
public static void main(String[]args){
    int num1 = 10;
    int num2 = 20;
    swap(num1,num2);

    System.out.println("num1="+num1);       // 10
    System.out.println("num2="+num2);          // 20
}

public static void swap(int a,int b){
    int temp = a;
    a = b ;
    b = temp;

    System.out.println("a="+a);     //  20
    System.out.println("b="+b);     // 10
}
```
在swap方法中，a、b的值进行交换，并不会影响num1、num2。因为a、b的值，只是从num1、num2复制过来的，也就是说，a、b相当于num1、num2的副本，副本如何修改都不会影响原件本身。

通过上面例子，我们已经知道了一个方法不能修改一个基本数据类型的参数，而对象引用作为参数就不一样，

**example2**
```java
public static void main(String[]args){
    int[]arr = {1,2,3,4,5};
    System.out.println(arr[0]);     // 1 
    change(arr);
    System.out.println(arr[0]);         //0 
}

public static change(int[]array){
    array[0] = 0;
}
```
array被初始化arr的拷贝也就是一个对象的引用，array和arr指向同一个数组的对象，外部对对象的改变会反映到对应的对象上。

**example  3**
```java
public class Test{
    public static void main(String []args){
        Student s1 = new Student("w")
        Student s2 = new Student("h");
        Test.swap(s1,s2);
        System.out.println("s1:" +s1.getName());  //w
        System.out.println("s2:" +s2.getName());  //h
    }

    public static void swap(Studnet x,Student y){
        Student temp = x;
        x = y;
        y = temp;
        System.out.println("x:" +x.getName());  // h
        System.out.println("y:" +y.getName());  // w
    }
}
```
方法并没有改变存储在变量 s1 和 s2 中的对象引用。swap 方法的参数 x 和 y 被初始化为两个对象引用的拷贝，这个方法交换的是这两个拷贝

**总结**
Java程序设计语言对对象采用的不是引用调用，实际上，对象引用是按值传递的

- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）。
- 一个方法可以改变一个对象参数的状态。
- 一个方法不能让对象参数引用一个新的对象。

## 重载 与 重写

**重载**

发生在同一个类中(或者父类和子类之间)，方法名必须相同，参数类型不同，个数不同，顺序不同，方法返回值和访问修饰符可以不同。

```java
StringBuilder s1= new StringBuilder();
StringBuilder s1= new StringBuilder("1");
```
重载就是同一个类中多个同名方法根据不同的传参来执行不同的逻辑处理。

**重写**
重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写

1. 返回值类型、方法名、参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类
2. 如果父类方法访问修饰符为`private/final/static`则子类不能重写该方法，但是被`static`修饰的方法能够再次被声明
3. 构造方法无法被重写

重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变

区别点 | 重载方法 | 重写方法
:--: | :---: | :---: 
发生范围 | 同一个类 | 子类
参数列表 | 必须修改 | 必须相同
返回类型 | 可修改 | 子类方法返回值类型应比父类方法返回值类型更小或相等
异常 | 可修改 | 子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等
访问修饰符 | 可修改 | 一定不能做更严格的限制(可降低限制)
发生阶段 | 编译期 | 运行期

**方法的重写要遵循“两同两小一大”**
- “两同”即方法名相同、形参列表相同；
- “两小”指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；
- “一大”指的是子类方法的访问权限应比父类方法的访问权限更大或相等。

如果方法的返回类型是 void 和基本数据类型，则返回值重写时不可修改。但是如果方法的返回值是引用类型，重写时是可以返回该引用类型的子类的

```java
public class Hero(){
    public String name(){
        return "超级英雄";
    }
}
public class SuperMan extends Hero{
    @Override
    public String name(){
        return "超人";
    }

    public Hero hero(){
        return new Hero();
    }
}

public class SuperSuperMan extends SuperMan{
    public String name(){
        return "超级超级英雄";
    }

    @Override
    public SuperMan hero(){
        return new SuperMan();
    }
}
```

## 深拷贝 vs 浅拷贝
1. 浅拷贝：对基本数据类型进行值传递，对引用数据类型，进行引用传递般的拷贝
2. 深拷贝：对基本数据类型进行值传递，对引用数据类型。创建一个新的对象，并复制其内容

![深拷贝vs浅拷贝](http://whh.plus:7007/images/2021/07/24/java-deep-and-shallow-copy.jpg)

## Java面向对象
**面向对象和面向过程的区别**
- 面向过程：面向过程性能比面向对象高。因为类调用时需要实例化，开销比较大，比较耗资源，所以当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux/Unix 等一般采用面向过程开发。但是，面向过程没有面向对象易维护、易复用、易扩展。
- 面向对象：面向对象易维护、易复用、易扩展。因为面向对象有封装、继承、多态性的特性。面向对象性能比面向过程低。

**成员变量与局部变量的区别**
1. 成员变量属于类，局部变量是在代码块或方法中定义的变量或是方法的参数。成员变量可以被public、private、static等修饰符修饰，而局部变量不能被访问控制修饰符及static修饰。都可被final修饰
2. 如果成员变量使用static修饰，那么成员变量属于类，如果没有使用static修饰，则属于实例，对象存在于堆内存，局部变量存在于栈内存
3. 成员变量是对象的一部分，随着对象的创建而存在，而局部变量随方法的调用而自动消失
4. 成员变量如果没有赋初值，则会自动以类型的默认值赋值(被final修饰的成员变量也必须显示赋值)，局部变量不会自动赋值

new创建对象实例(对象实例在堆内存中)，对象引用指向对象实例(对象引用放在栈内存中)

对象的相等，比的是内存中存放的内容是否相等。而引用相等，比较的是他们指向的内存地址是否相等。

**一个类的构造方法**作用：完成对类对象的初始化工作；即使没有声明构造方法也会有默认的不带参数的构造方法。

**构造方法特点**
1. 名字与类名相同
2. 没有返回值，但不能用void声明构造函数
3. 生成类的对象时自动执行，无需调用

构造方法不能被重写，但可以被重载，一个类可以有多个构造函数

## 面向对象 三大特征
**封装**
封装：把一个对象的状态信息(属性)隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但可提供一些被外界访问的方法来操作属性。
```java
public class Student {
    private int id;//id属性私有化
    private String name;//name属性私有化
    //获取id的方法
    public int getId() {
        return id;
    }
    //设置id的方法
    public void setId(int id) {
        this.id = id;
    }
    //获取name的方法
    public String getName() {
        return name;
    }
    //设置name的方法
    public void setName(String name) {
        this.name = name;
    }
}
```
**继承**
不同类型的对象，相互之间经常有一定数量的共同点，同时每个对象还定义了额外的特性使之不同。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可用父类的功能，但不能选择性继承父类。通过使用继承，可以快速创建新的类，可以提高代码的重用，程序可维护性，节省创建新类的时间，提高开发效率。

- 子类拥有父类对象所有属性和方法(包括私有属性和私有方法)，但父类中的私有属性和私有方法子类无法访问，只是拥有
- 子类可以拥有自己的属性和方法，即子类对父类进行扩展
- 子类可以用自己的方式实现父类方法

**多态**
表示一个对象具有多种状态，具体表现为父类的引用指向子类的实例
**多态特点**
1. 对象类型和引用类型之间具有继承(类)/实现(接口)的关系
2. 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定
3. 多态不能调用"只在子类存在但在父类不存在"的方法
4. 如果子类重写了父类的方法，真正执行的是子类覆盖方法，如果子类没有覆盖，执行的是父类方法


## String、StringBuffer、StringBuilder区别
**String**:使用final关键字修饰字符数组来保存字符串，`private final char value[]`，String对象不可变，Java9之后使用byte数组存储字符串

StringBuffer和StringBuilder都继承AbstractStringBuilder类，在AbstractStringBuilder中也是使是使用字符数组存储字符串，但没有使用final修饰，所以这两个对象是可变的。

StringBuilder 和 StringBuffer的构造方法都是调用父类的AbstractStringBuilder实现的。

**线程安全性**
String：对象是不可变的，线程安全

StringBuffer：对方法或者对调用的方法加了同步锁，所以线程安全

StringBuilder：没有对方法进行加同步锁，所以是非线程安全

**性能**
String：每次对String类型进行改变时，都会生成一个新的String对象，然后将指针指向新的String对象。

StringBuffer:每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用

StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结**：
1. 操作少量的数据：适用String
2. 单线程操作字符串缓冲区下操作大量数据：适用StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据：适用StringBuffer

## Object类的常见方法总结
Object类是一个特殊的类，是所有类的父类，主要提供以下11个方法：
- `public final nativa Class<?> getClass() //native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写`
- `public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap`
- `public boolean equals(Object obj) //用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等`
- `protected native Object clone() throws CloneNotSupportedException  //native方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象x，表达式x.clone()!=x 为true。x.clone().getClass()==x.getClass()为true。Object本身没有实现Cloneable接口，所以不重写clone()方法并且进行调用的话会法神CloneNotSupportedException异常`
- `public String toString() //返回类的名字@实例的哈希码的16进制的字符串。建议所有子类都重写这个方法`
- `public final native void notify()  //native方法，并且不能重写，唤醒一个在此对象监视器上等待的线程(监视器相当于锁的概念)如果有多个线程在等待只会任意唤醒一个`
- `public final native void notifyAll() //native方法，不能重写，跟notify一样，唯一的区别就是唤醒在此对象监视器上等待的所有线程，而不是一个线程`
- `public final native void wait(long timeout) throws InterruptedException //native方法，不能重写，暂停线程的运行。注意sleep方法并没有释放锁，但wait方法释放了锁。timeout是等待时间`
- `public final native void wait(long timeout,int nanos) throws InterruptedException  //多nanos参数，这个参数表示额外时间(以毫微秒为单位，范围是0-999999).所以超时的时间还需要加上nanos毫秒`
- `public final native void wait() throws InterruptedException //跟之前2个wait方法一样，只不过该方法一直等待，没有超时概念`
- `protected void finalize() throws Throwable{} //实例被垃圾回收器回收的时候触发的操作` 

## 反射
反射是框架的灵魂，主要是因为它赋予了我们在运行时分析类以及执行类中方法的能力，通过反射可以获取任何一个类的所有属性和方法，并且还可以调用这些方法和属性

**反射机制优缺点**
- 优点：让代码更灵活，为各种框架提供开箱即用的功能提供便利
- 缺点：让我们在运行时有了分析操作类的能力，增加安全问题。比如可以无视泛型参数的安全检查(泛型参数检查发生在编译时)。另外反射的性能也要稍差一点。

**反射应用场景**
像Spring/Spring boot、Mybatis等框架中都大量使用了反射机制，这些框架也大量使用了动态代理，而动态代理的实现也依赖反射

通过JDK实现动态代理的示例代码，使用反射类Method来调用指定的方法。
```java
public class DebugInvocationHandler implements InvocationHandler{
    /**
    代理类中的真实对象
    */
    private final Object target;

    public DebugInvocationHandler(Object target){
        this.target = target;
    }

    public Object invoke(Object proxy ,Method method, Object[]args) throws InvocationTargetException,IllegalAccessException{
        System.out.println("before method" + method.getName());
        Object result = method.invoke(target,args);
        System.out.println("after method" + method.getName());
        return result;
    }
}
```
注解的实现也用到了反射。基于反射分析类，获取到类/属性/方法/方法的参数上的注解。

## 异常
**Java异常类层次结构图**
![异常类层次结构图](http://whh.plus:7007/images/2021/07/26/JavaE5BC82E5B8B8E7B1BBE5B182E6ACA1E7BB93E69E84E59BBE.png)
![](http://whh.plus:7007/images/2021/07/26/JavaE5BC82E5B8B8E7B1BBE5B182E6ACA1E7BB93E69E84E59BBE2.png)
在Java中，所有的异常都有一个共同的祖先`java.lang`包中的`Throwable`类。`Throwable`类有两个重要的子类`Exception(异常)`和`Error(错误)`。`Exception`能被程序本身处理`try-catch`，`Error`是无法处理的(只能尽量避免)

`Exception`和`Error`二者都是Java异常处理的重要子类，各自都包含大量子类。
- Exception：程序本身可以处理的异常，可以通过`catch`来进行捕获。Exception又可以分为 受检查异常(必须处理) 和 不受检查异常(可以不处理)
- Error：属于程序无法处理的错误，没法通过catch来进行捕获。例如 Java虚拟机运行错误、虚拟机内存不够错误、类定义错误等。这些异常发生时，Java虚拟机(JVM)一般会选择线程终止。

**受检查异常**
Java代码在编译过程中，如果受检查异常没有被`catch/throw`处理的话，就没办法通过编译。

```java
public static void main(String[]args) throws IOException{
    FileInputStream fis = null;
    fis = new FileInputStream("xx/xx.txt");
    int k;
    while(k =fis.read() != -1){
        System.out.println((char) k);
    }
    fis.close();
}
```
除了`RuntimeException`及其子类以外，其他的Exception类及其子类都属于受检查异常。常见的受检查异常有：IO相关异常、ClassNotFoundException、SQLException,...

**不受检查异常**
Java代码在编译过程中，即使不处理不受检查异常也可以正常通过编译

`RuntimeException`及其子类都统称为非受检查异常。例如`NullPointerException、NumberFormatException(字符串转换为数字)、ArrayIndexOutOfBoundsException(数组越界)、ClassCastException(类型转换错误)、ArithmeticException(算术错误)`等

**Throwable类常用方法**
- public String getMessage():返回异常发生时的简要描述
- public String toString()：返回异常发生时的详细信息
- public String getLocalizedMessage()：返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以生成本地化信息。如果子类没有覆盖该方法，则方法返回的信息与getMessage()返回的结果相同
- public void printStackTrace():在控制台上打印Throwable对象封装的异常信息

**`try-catch-finally`**
- try块:用于捕获异常。其后可接0个或多个`catch`块，如果没有`catch`,则必须跟一个`finally`块
- catch块：用于处理try捕获到的异常
- finally块：无论是否捕获或处理异常，finally块里语句都会被执行。当try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行

**以下3中特殊情况下，finally块不会被执行**
1. 在try块或finally块中用了System.exit(int)退出程序。但是如果System.exit(int)在异常语句之后，finally还是会被执行
2. 程序所有的线程死亡
3. 关闭CPU

注意：当try语句和finally语句中都有return语句时，在方法返回之前，finally语句的内容将被执行，并且finally语句的返回值将会覆盖原始的返回值。如下：
```java
public class Test{
    public static int test(int value){
        try{
            return value * value;
        }finally{
            if(value == 1)
                return 0;
        }
    }
}
```
如果调用 test(2)，返回值将是 0，因为 finally 语句的返回值覆盖了 try 语句块的返回值。

**使用try-with-resources 来代替 try-catch-finally**
1. 适用范围(资源的定义) ：任何实现java.lang.AutoCloseable或者java.io.Closeable的对象
2. 关闭资源和finally块的执行顺序：在try-with-resource语句中，任何catch和finally块在声明的资源关闭后运行

《Effective Java》中明确指出：
> 面对必须要关闭的资源，我们总是应该优先使用try-with-resources 而不是try-finally。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。try-with-resources语句让我们更容易编写必须要关闭的资源代码，若采用try-finally则几乎做不到这点

Java中类似于InputStream、OutputStream、Scanner、PrintWriter等的资源都需要调用close()来手动关闭，一般情况下我们是通过try-catch-finally语句实现这个需求，如下：
```java
//读取文本文件的内容
Scanner scanner = null;
try{
    scanner = new Scanner(new File("d://XXX.txt"));
    while(scanner.hasNext()){
        System.out.println(scanner.nextLine());
    }
}catch(FileNotFoundException e){
    e.printStackTrace();
}finally{
    if(scanner!=null)
        scanner.close();
}
```
使用Java7之后的try-with-resources语句改造上面的代码
```java

try (Scanner scanner = new Scanner(new File("d://XXX.txt"))){
    while(scanner.hasNext()){
        System.out.println(scanner.nextLine());
    }
}catch(FileNotFoundException e){
    e.printStackTrace();
}
```
当然多个资源需要关闭的时候，使用try-with-resources实现起来也非常简单，如果你还是用try-catch-finally可能会带来很多问题

通过使用分号分隔，可以在try-with-resources块声明多个资源
```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
             BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))){
                int b;
                while ((b = bin.read()) != -1) {
                    bout.write(b);
                }
             } catch (IOException e){
                 e.printStackTrace();
             }
```

## I/O流
**什么是序列化？什么是反序列化？**
如果我们需要持久化Java对象比如将Java对象保存在文件中，或者在网络传输Java对像，这些场景都需要用到序列化。

- 序列化：将数据结构或对象转换成二进制字节流的过程
- 反序列化：将在序列化过程中所生成的二进制字节流的过程转换成数据结构或对象的过程

对于Java这种面向对象编程语言来说，序列化的都是对象(Object)也就是实例化后的类(Class),但是在C++这种半面向对象的语言中，struct(结构体)定义的是数据结构类型，而class对应的是对象类型。

> 序列化在计算机科学的数据处理中，是指将数据结构或对象状态转换成为可取用格式(例如存成文件，存于缓冲，或经由网络中发送)，以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始对象相同语义的副本。对于许多对象，像是使用大量引用的复杂对象，这种序列化重建的过程并不容易。面向对象中的对象序列化，并不概括之前原始对象所关系的函数。这种过程也称为对象编组。从一系列字节提取数据结构的反向操作，是反序列化

**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**

![序列化与反序列化](http://whh.plus:7007/images/2021/07/27/a478c74d-2c48-40ae-9374-87aacf05188c.png)

**Java序列化中如果有些字段不想进行序列化，怎么办？**
对于不想进行序列化的变量，使用`transient`关键字修饰

`transient`关键字的作用是：阻止实例中那些用此关键字修饰的变量序列化；当对象被反序列化时，被`transient`修饰的变量值不会被持久化和恢复。`transient`只能修饰变量，不能修饰类和方法

**获取用键盘输入常用的两种方法**
1. 通过Scanner
```java
Scanner input = new Scanner(System.in);
String s = input.nextLine();
input.close(); 
```
2. 通过BufferedReader
```java
BufferedReadr input = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```

Java中IO流分为几种
- 按照流的流向分：输入流和输出流
- 按照操作单元划分：字节流和字符流
- 按照流的角色划分：节点流和处理流

Java流共涉及40多个类。
- InputStream/Reader：所有的输入流的基类，前者是字节输入流，后者是字符输入流
- OutputStream/Writer：所有的输出流的基类，前者是字节输出流，后者是字符输出流

按操作方式分类结构图：
![按操作方式分类结构图](http://whh.plus:7007/images/2021/07/27/IO-E6938DE4BD9CE696B9E5BC8FE58886E7B1BB.jpg)

按操作对象分类结构图：
![按操作对象分类结构图](http://whh.plus:7007/images/2021/07/27/IO-E6938DE4BD9CE5AFB9E8B1A1E58886E7B1BB.jpg)

**既然有了字节流，为什么还要有字符流？**
不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么I/O操作要分为字节流操作和字符流操作呢？

字符流是由Java虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时的，并且，如果我们不知道编码类型就很容易出现乱码问题，所以I/O流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作，如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的化使用字符流比较好。

**参考**
[JavaGuide-Java基础](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86?id=%e4%bd%bf%e7%94%a8-try-with-resources-%e6%9d%a5%e4%bb%a3%e6%9b%bftry-catch-finally)
