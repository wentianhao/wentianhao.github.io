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

@Configuration：等同于Spring的XML配置文件；使用Java代码可以检查类型安全。

@EnableAutoConfiguration:自动配置

@Autowired：自动导入依赖的Bean

@Resource(name="name",type="type")：没有括号内容的话，默认byName。与@Autowired干类似的事

@Bean:用@Bean标注方法等价于XML中配置的bean

@ComponentScan: 表示将该类自动发现扫描组件。个人理解相当于，如果扫描到有@Component、@Controller、@Service等这些注解的类，并注册为Bean,可以自动收集所有的spring组件，包括@Configuration。经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入，如果没有配置的话，SpringBoot会扫描启动类所在包下以及子包下的使用了@Service，@Repository等注解的类。

@Service:一般用于修饰service层的组件

@Repository:使用@Repository注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，同时也不需要为它们提供XML配置项

### Controller控制层注解
@Controller:用于定义控制器类，在spring项目中由控制器负责将用户发出来的URL请求转发到对应的服务接口(service层)，一般这个注解在类中，通常方法需要配合注解@RequestMapping一起使用   

@ResponseBody: 表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，用于构建RESTful的api。在使用RequestMapping后，返回值通常解析为跳转路径，加上@ResponseBody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中，比如异步获取json数据，加上@ResponseBody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用。

@RestController：用于标准控制层组件(如structs中的action)，@ResponseBody和@Controller的合集

@RequestMapping:提供路由信息，负责URL到Controller中的具体函数的映射,@RequestMapping("/path")表示该控制器处理所有"/path"的URL请求,RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。

### JPA注解
@Entity: @Table(name=""):表明这是一个实体类，一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@Id:表示该属性为主键

@GeneratedValue(strategy=GenerationType.SEQUENCE,generator="repair_seq"):表示主键生成策略是sequence(可以是auto、IDENTITY、native等，Auto表示可在多个数据库间切换),指定sequence的名字是repair_seq。

@Column:如果字段名与列名相同，则可以省略