---
title: SpringBoot注解
date: 2021-07-11 10:53:48
tags: SpringBoot
categories: SpringBoot
---
使用注解的优势：
1. 采用Java代码，不需要配置繁杂的xml文件
2. 在配置中也可享受面向对象带来的好处
3. 类型安全对重构可以提供良好的支持
4. 减少复杂配置文件的同时亦能享受到springIoC容器提供的功能
<!-- more -->

@SpringBootApplication：申明让spring boot自动给程序进行必须的配置，这个配置等同于：@Configuration，@EnableAutoConfiguration和@ComponentScan三个配置

### Controller控制层注解
@Controller:用于定义控制器类，在spring项目中由控制器负责将用户发出来的url   

### JPA注解
@Entity: @Table(name=""):表明这是一个实体类，一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@Id:表示该属性为主键

@GeneratedValue(strategy=GenerationType.SEQUENCE,generator="repair_seq"):表示主键生成策略是sequence(可以是auto、IDENTITY、native等，Auto表示可在多个数据库间切换),指定sequence的名字是repair_seq。

@Column:如果字段名与列名相同，则可以省略