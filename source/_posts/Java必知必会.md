---
title: Java必知必会
katex: false
tags: java
categories: java
abbrlink: 39124
date: 2021-08-14 12:18:13
---

概括Java基础、Java异常、Java集合、Java并发、Java JVM、SSM框架、MySQL、Redis、计算机网络、操作系统、消息队列与分布式
<!-- more  -->

## Java基础 44道
1. 解释下什么是面向对象？面向对象和面向过程的区别？
- 面向对象的编程是以对象为中心，以消息为驱动。

2. 面向对象的三大特性？分别解释下？
- 封装、继承、多态

3. JDK、JRE、JVM 三者之间的关系？
- JDK = JRE + Javadoc + jdb
- JRE = JVM + 一些类库

4. 重载和重写的区别？
- 重载是同一个方法根据输入不同，做出不同的处理
- 重写是子类继承父类相同方法，覆盖父类方法

5. Java 中是否可以重写一个 private 或者 static 方法？
- 不能。方法覆盖属于运行时动态绑定的，static方法是编译时静态绑定的。private修饰的变量或方法只能在当前类中使用，其他类继承无法访问private变量或方法，无法覆盖。

6. 构造方法有哪些特性？

7. 在 Java 中定义一个不做事且没有参数的构造方法有什么作用？
8. Java 中创建对象的几种方式？
9.  抽象类和接口有什么区别？
10. 静态变量和实例变量的区别？
11. short s1 = 1；s1 = s1 + 1；有什么错？那么 short s1 = 1; s1 += 1；呢？有没有错误？
12. Integer 和 int 的区别？
13. 装箱和拆箱的区别
14. switch 语句能否作用在 byte 上，能否作用在 long 上，能否作用在 String 上？
15. final、finally、finalize 的区别
16. == 和 equals 的区别？
17. 两个对象的 hashCode() 相同，则 equals() 也一定为 true 吗？
18. 为什么重写 equals() 就一定要重写 hashCode() 方法？
19. & 和 && 的区别？
20. Java 中的参数传递时传值呢？还是传引用？
21. Java 中的 Math.round(-1.5) 等于多少？
22. 如何实现对象的克隆？
23. 深克隆和浅克隆的区别？
24. 什么是 Java 的序列化，如何实现 Java 的序列化？
25. 什么情况下需要序列化？
26. Java 的泛型是如何工作的 ? 什么是类型擦除 ?
27. 什么是泛型中的限定通配符和非限定通配符 ?
28. List 和 List 之间有什么区别 ?
29. Java 中的反射是什么意思？有哪些应用场景？
30. 反射的优缺点？
31. Java 中的动态代理是什么？有哪些应用？
32. 怎么实现动态代理？
33. static 关键字的作用？
34. super 关键字的作用？
35. 字节和字符的区别？
36. String 为什么要设计为不可变类？
37. String、StringBuilder、StringBuffer 的区别？
38. String 字符串修改实现的原理？
39. String str = "i" 与 String str = new String("i") 一样吗？
40. String 类的常用方法都有那些？
41. final 修饰 StringBuffer 后还可以 append 吗？
42. Java 中的 IO 流的分类？说出几个你熟悉的实现类？
43. 字节流和字符流有什么区别？
44. BIO、NIO、AIO 有什么区别？