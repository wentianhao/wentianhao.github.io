---
title: 反射机制
katex: false
tags: java
categories: java
abbrlink: 40365
date: 2021-07-30 09:03:10
---

详细学习一下Java的反射机制
<!-- more -->

反射使我们在运行时分析类以及执行类中方法的能力，通过反射可以获得任意一个类的所有属性和方法，还可以调用这些属性和方法

Spring/Spring Boot、MyBatis等框架都大量使用反射机制，也大量使用了动态代理，动态代理的实现也依赖反射

通过JDK实现动态代理的示例代码，其中用到了反射类Method来调用指定的方法
```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
    代理类中的真实对象
    */
    private final Object target;

    public DebugInvocationHandler(Object target){
        this.target = target;
    }

    public Object invoke(Object proxy,Method method,Object[]args) throws InvocationTargetException,IllegalAccessException{
        System.out.println("before method" + method.getName());
        Object result = method.invoke(target,args);
        System.out.println("after method" + method.getName());
        return result;
    }
}
```

注解的实现也用到了反射，基于反射分析类，获取类/属性/方法/方法的参数上的注解。

## 反射机制的优缺点
- 优点：可以让代码更灵活，为各种框架提供开箱即用的功能提供了便利
- 缺点：让我们在运行时有了分析操作类的能力，增加了安全问题。比如无视泛型参数的安全检查(泛型参数的安全检查发生在编译时)。反射的性能稍差一点，对于框架影响不大。

## 反射实战
**获取class对象的四种方法**
如果动态获取到这些信息，需要依靠class对象。class类对象将一个类的方法，变量等信息告诉运行的程序。Java提供四种方法获取class对象

1. 知道具体类的情况下使用
```java
Class alunbarClass = TargetObject.class;    //通过此方法获取class对象不会初始化
```
2. 通过Class.forName()传入类的路径获取
```java
Class alunbarClass = Class.forName("com.xxx.TargetObject");
```
3. 通过对象实例`instancce.getClass()`获取
```java
TargetObject o = new TargetObject();
Class alunbarClass = o.getClass();
```
4. 通过类加载器`xxxClassLoader.loadClass()`传入类路径获取
```java
Class clazz = ClassLoader.loadClass("com.xxx.TargetObject");
```
通过类加载器获取Class对象不会进行初始化，意味着不进行包括初始化等一系列步骤，静态块和静态对象不会得到执行

## 反射的一些基本操作
1. 创建一个要使用反射操作的类`TargetObject`
```java
package com.example.wanheo;

public class TargetObject {
    private String value;

    public TargetObject(){
        value = "wanheo";
    }

    public void publicMethod(String s){
        System.out.println("i " + s);
    }

    private void privateMethod(){
        System.out.println("value is" + value);
    }
}
```
2. 使用这个反射操作这个类的方法和参数
```java
package com.example.wanheo;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[]args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        /**
         * 获取TargetObject类的class对象并创建TargetObject类实例
         */
        Class<?> targetClass = Class.forName("com.example.wanheo.TargetObject");
        TargetObject targetObject =  (TargetObject)targetClass.newInstance();

        /**
         * 获取所有类中所有定义的方法
         */
        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods){
            System.out.println(method.getName());
        }

        /**
         * 获取指定方法并调用
         */
        Method publicMethod = targetClass.getDeclaredMethod("publicMethod", String.class);
        publicMethod.invoke(targetObject,"1");
        /**
         * 获取指定参数并对参数进行修改
         */
        Field field = targetClass.getDeclaredField("value");
        //为了对类中参数进行修改取消安全检查
        field.setAccessible(true);
        field.set(targetObject,"2");

        /**
         * 调用private方法
         */
        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        //为了调用private方法取消安全检查
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}

```
输出内容：
```bash
publicMethod
privateMethod
i 1
value is 2
```