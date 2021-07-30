---
title: 代理模式
katex: false
tags: java
categories: java
abbrlink: 48003
date: 2021-07-30 09:44:12
---

设计模型之代理模式
<!-- more -->
## 代理模式
代理模式是一种比较好理解的设计模式。**使用代理对象来替代对真实对象的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能**。代理模式的主要作用是 扩展目标对象的功能，比如在目标对象的某个方法执行前后可以增加一些自定义的操作。

例如 ： 你找小红来帮你问话，小红就可看作你的代理对象，代理的行为(方法)就是问话。

代理模式有静态代理和动态代理两种实现方式

## 静态代理
**静态代理中，我们对目标对象的每个方法的增强都是手动完成的**，非常不灵活，接口一旦新增方法，目标对象和代理对象都要进行修改，且麻烦，需要对每个目标类都单独写一个代理类。

从JVM层面来说，静态代理在编译时就将接口、实现类、代理类这些变成了一个个实际的class文件

静态代理的实现步骤
1. 定义一个接口及其实现类
2. 创建一个代理类同样实现这个接口
3. 将目标对象注入进代理类，然后在代理类的对应方法调用目标类中的对应方法。这样就可以通过代理类屏蔽对目标对象的访问，并且在目标方法执行前后做一些自己想做的事

**demo**
1. 定义发送短信的接口
```java
public interface SmsService{
    String send(String message);
}
```
2. 实现发送短信接口的实现类
```java
public class SmsServiceImpl implements SmsService{
    public String send(String message){
        System.out.println("send message" + message);
        return message;
    }
}
```
3. 创建代理类并同样实现发送短信的接口
```java
public class SmsProxy implements SmsService{
    private final SmsService smsService;

    public SmsProxy(SmsService smsService){
        this.smsService = smsServicel
    }

    @Override
    public String send(String message){
        //使用方法之前，添加自己的操作
        System.out.println("before method send()");
        smsService.send(message);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method send()");
        return null;
    }
}
```
4. 实际使用
```java
public class Main(){
    public static void main(String[]args){
        SmsService smsService = new SmsServiceImpl();
        SmsProxy smsProxy = new SmsProxy(smsService);
        smsProxy.send("java");
    }
}
```
输出结果
```bash
before method send()
send message java
after method send()
```

##  动态代理
相比于静态代理，动态代理更灵魂，不需要针对目标类单独创建一个代理类，并且也不需要必须实现接口，可以直接代理实现类(CGLIB动态代理机制)

从jvm角度，动态代理是在运行时动态生成类字节码，并加载到jvm中

Spring AOP、RPC框架的实现都依赖了动态代理。

动态代理在框架中几乎是必用的一门技术。动态代理的实现方法：JDK动态代理，CGLIB动态代理等等

### JDK动态代理机制
在Java动态代理机制中，InvocationHandler接口和Proxy类是核心，Proxy类使用频繁最高的方法是`newProxyInstance()`,这个方法主要是生成一个代理对象
```java
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interface,InvocationHandler h) throws IllegalArgumentException{
    ....
}
```
三个参数
1. loader:类加载器，用于加载代理对象
2. interface：被代理类是实现的一些接口
3. h：实现了InvocationHandler接口的对象

要实现动态代理的话，还必须实现 InvocationHandler来自定义出来逻辑。当动态代理对象调用一个方法时候，这个方法的调用给就会被转发到实现InvocationHandler接口类的invoke方法来调用
```java
public interface InvocationHandler{
    /**
    当你使用代理对象调用方法时实际会调用到这个方法
    */
    public Object invoke(Object proxy,Method method,Object[]args) throws Throwable;
}
```

invoke()方法有下面三个参数：
1. proxy：动态代理生成的代理类
2. method：与代理类对象调用的方法相对应
3. args：当前method方法的参数

通过proxy类的newProxyInstance()创建的代理对象在调用方法的时候，实际上调用到实现InvocationHandler接口的类的invoke()方法。

**JDK动态代理类使用步骤**
1. 定义一个接口及其实现类
2. 自定义InvocationHandler并重写invoke()方法，在invoke()方法中调用原生方法(被代理类的方法)并自定义一些处理逻辑
3. 通过Proxy.newProxyInstance(ClassLoader loader,Class<?> interface,InvocationHandler h)方法创建代理对象

1. 定义发送短信的接口
```java
public interface SmsService{
    String send(String message);
}
```
2. 实现发送短信的接口
```java
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```
3. 定义一个JDK动态代理类
```java
public class DebugInvocationHandler implements InvocationHandler{
    private final Object target;
    class DebugInvocationHandler(Object target){
        this.target = target;
    }

    public Object invoke(Object proxy,Method method,Object[]args) throws InvocationTargetException,IllegalAccessException{
         //调用方法之前，我们可以添加自己的操作
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target,args);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method " + method.getName());
        return result;
    }
}
```
invoke():当动态代理对象调用原生方法时，实际最终调用的是invoke()方法，然后invoke()方法代替去调用被代理对象的原生方法
4. 获取代理对象的工厂类
```java
public class JdkProxyFactory{
    public static Object getProxy(Object target){
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(), //目标类的类加载
            target.getClass().getInterfaces(),  //代理需要实现的接口，可指定多个
            new DebugInvocationHandler(target)  //代理对象对应的自定义 InvocationHandler
        );
    }
}
```
getProxy():主要通过对Proxy.newProxyInstance()方法获取某个类的代理对象

**实际使用**
```java
SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
smsService.send("java");
```
结果
```bash
before method send
send message:java
after method send
```

## CGLIB动态代理机制
jdk动态代理有一个最致命的问题是其只能代理实现了接口的类，为了解决这个问题，可以使用CGLIB动态代理机制来避免

CGLIB(code generation library)是一个基于ASM的字节码生成库，允许在运行时对字节码进行修改和动态生成.CGLIB通过继承的方式实现代理。Spring中的AOP模块中，如果目标对象实现了接口，则默认采用JDK动态代理，否则就是采用CGLIB动态代理



