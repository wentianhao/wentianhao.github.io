---
title: java基础
date: 2021-07-21 09:38:34
tags:
categories:
---

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
