---
title: spring cloud mybaits 微服务增删改查(一)
tags:
  - spring cloud
  - mybatis
categories:
  - spring cloud
abbrlink: 25536
date: 2020-01-13 17:23:39
---

IDE：IDEA 2017.03

**1.构建spring cloud聚合项目**

1)新建maven项目

不需要使用模板，直接next，填写项目名称继续next,最后finish
<!-- more -->
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925111020451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

2）构建子模块

删除掉父模块中的src文件夹，新建子模块，最后层次结构如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925111706101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

在项目工程上右键new->Module，一直next就行，注意一下继承的父模块。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925112426157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

其他模块也是如此新建。

父模块的pom.xml：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925112659123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

子模块的pom.xml:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925112727798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

**2.配置注册中心**

- 配置spring boot spring cloud等依赖。公共依赖可放在父模块,子模块可以调用父模块的pom.xml

```yml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion
    
    <groupId>com.springcloud.demo</groupId>
    <artifactId>mybatis</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <!--子模块设置-->
    <modules>
        <module>mybatisregister</module>
        <module>mybatisproduce</module>
        <module>mybatisconsumer</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <!--eureka server -->
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <!-- spring boot test-->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.M7</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

- 手动新建application.yml（文件名不可随意更改）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925113449653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

```yml
# 端口号
server:
port: 8088
#eureka配置
eureka:
instance:
#注册中心ip地址
    hostname: localhost
client:
    #是否注册到eureka
    register-with-eureka: false
    #是否检测服务信息
    fetch-registry: false
    #注册地址
    service-url:
    defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

- 新建运行类application

@EnableEurekaServer 注解，声明这是一个Eureka Server

```java
package com.springcloud.register;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class mybatisRegister_Application {

    public static void main(String[] args){
        SpringApplication.run(mybatisRegister_Application.class,args);
    }

}
```

**3.配置服务提供者**

在另一个子模块中进行配置

- 添加application.yml和application

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925114804773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

```yml
eureka:
client:
#注意！这里注册中心地址，要把服务注册到注册中心
    service-url:
    defaultZone: http://localhost:8088/eureka/
#服务端口号
server:
port: 8089
spring:
#本服务的注册到注册中心的别名
application:
    name: service-member
jpa:
    generate-ddl: false
    show-sql: true
    hibernate:
    ddl-auto: none
datasource:
    url: jdbc:mysql://127.0.0.1:3306/ssmdemo?serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
mapper-locations: classpath*:mappers/*.xml
```

这里使用的注解是@EnableEurekaClient

```java
package com.springcloud.produce;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@MapperScan("com.springcloud.produce.mapper")
public class mybatisProduce_Application {
    public static void main(String[] args){
        SpringApplication.run(mybatisProduce_Application.class,args);
    }
}
```

构建测试controller

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925115250599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

```java
package com.springcloud.produce.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class testController {
    @RequestMapping("/test")
    public String getUserList(){
        return "hello eureka";
    }
}
```

运行注册模块和生产微服务模块 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925115709117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925115814903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

**4.配置服务消费者**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925145616621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)
application.yml:

```yml
eureka:
client:
#注意！ 这里注册中心地址，要把服务注册到注册中心
    service-url:
    defaultZone: http://localhost:8088/eureka/
server:
#服务端口号
port: 8087
spring:
#本服务注册到注册中心的别名
application:
    name: service-order
```

application.class

这里需要将RestTemplate注入到springboot中，然后开启负载均衡

```java
package com.springcloud.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages = "com.springcloud.consumer.util")
public class mybatisConsumer_Application {
    public static void main(String[] args){
        SpringApplication.run(mybatisConsumer_Application.class,args);
    }

    /**
    * @Bean，是注入springboot
    * @LoadBalanced，开启负载均衡，开启后，restTemplate.getForObject()里的是serviceId(service-member)
    * @LoadBalanced,不开启时，restTemplate.getForObject()里面的时ip地址（127.0.0.1）
    */
    @Bean
    @LoadBalanced
    RestTemplate restTemplate(){
        return new RestTemplate();
    }

}
```

- 方法一：通过spring 提供的用于访问Rest服务的客户端

```java
package com.springcloud.consumer.controller;

import com.springcloud.consumer.util.MemberApiFeign;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class orderController {
// 自动注入RestTemplate类
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private MemberApiFeign memberApiFeign;

    //第一种方法通过Spring提供的用于访问Rest服务的客户端
    @RequestMapping("/getOrder")
    public String getOrder(){
        //这里放的url，使用的是服务提供者的别名+方法名
        String result = restTemplate.getForObject("http://service-member:8089/test",String.class);
        return result;
    }

    //第二种方法通过feign声明web服务客户端调用
    @RequestMapping("/getFeignOrder")
    public String getFeignOrder(){
        return memberApiFeign.Test();
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925144945696.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

- 方法二：通过feign声明web服务客户端调用原理

创建调用接口 FeignUtil.java

```java
package com.springcloud.consumer.util;

import com.springcloud.consumer.entity.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@FeignClient(name = "service-member")
public interface FeignUtil {
    @RequestMapping("/test")
    public String Test();

    @RequestMapping("/list")
    public List<User queryUserAll();

    @RequestMapping("/list")
    List<User getLists();

    @GetMapping("/{id}")
    public User queryUserById(@PathVariable("id") String id);

    @GetMapping("/addUser/{id}")
    public String addUser(@PathVariable("id") String id);

    @GetMapping("/deteleUser/{id}")
    public String deleteUser(@PathVariable("id") String id);

    @GetMapping("/updateUser/{id}")
    public String updateUser(@PathVariable("id") String id);
}
```

controller层调用↑↑↑代码里已贴。运行结果一样。

[下一篇实现mybatis的增删改查并注册到注册中心中，供消费者调用](https://blog.csdn.net/www_indows/article/details/101368222)。

[源码](https://download.csdn.net/download/www_indows/11818391)