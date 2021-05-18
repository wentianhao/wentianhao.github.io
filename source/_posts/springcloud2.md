---
title: spring cloud mybaits 微服务增删改查(二)
date: now()
tags: [spring cloud, mybatis]
categories: [spring cloud]
---


[在上一篇的基础上增加增删改查功能](https://blog.csdn.net/www_indows/article/details/101204585)

**创建数据库：**

```sql
CREATE DATABASE ssmdemo;
```
<!-- more -->
**创建表：**

```sql
DROP TABLE IF EXISTS tb_user;
CREATE TABLE tb_user (
id char(32) NOT NULL,
user_name varchar(32) DEFAULT NULL,
password varchar(32) DEFAULT NULL,
name1 varchar(32) DEFAULT NULL,
age int(10) DEFAULT NULL,
sex int(2) DEFAULT NULL,
birthday date DEFAULT NULL,
created datetime DEFAULT NULL,
updated datetime DEFAULT NULL,
PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**插入数据**

```sql
INSERT INTO ssmdemo.tb_user ( user_name, password, name1, age, sex, birthday, created, updated) 
VALUES ( ‘www’, ‘123456’, ‘我’, ‘22’, ‘1’, ‘2019-09-25’, sysdate(), sysdate());
INSERT INTO ssmdemo.tb_user ( user_name, password, name1, age, sex, birthday, created, updated) 
VALUES ( ‘ss’, ‘123456’, ‘哇’, ‘22’, ‘1’, ‘2019-09-25’, sysdate(), sysdate());
```

模块结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925155145677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

**entity层：**

```java
public class User {
    private String id;
    private String user_name;
    private String password;
    private String name1;
    private Integer age;
    private Integer sex;
    private Date birthday;
    private String created;
    private String updated;
    
/*setter and getter */
...
}
```

**mybatis**

引入依赖

```yml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.8</version>
</dependency>
```

**mybatis配置：**

1.全局配置：mybatis-config.xml

2.映射文件:  UserMapper.xml

**mybatis-config.xml:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 根标签 -->
<!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
<configuration>
    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="test">
        <!-- id：唯一标识 -->
        <environment id="test">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC" />
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/ssmdemo?serverTimezone=UTC" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mappers/UserMapper.xml"/>
    </mappers>
</configuration>
```

**UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--mappers：根标签，namespace：命名空间，随便写，在同一个命名空间下保持唯一-->
<mapper namespace="com.springcloud.produce.mapper.UserMapper">
<!--resultType:sql语句查询结果集的封装类型，tb_user即为数据库中的表-->
<!--resultMap标签，是为了映射select查询出来结果的集合，主要作用是将实体类中的字段和数据库表中的字段进行关联映射-->
<!--当实体类中的字段与数据库表中的字段相同时，可以将resultMap标签中的关联关系忽略不写-->

    <!--使用别名-->
    <select id="queryUserById" resultType="com.springcloud.mybatis.pojo.User">
        SELECT
        tuser.id as id,
        tuser.user_name as user_name,
        tuser.password as password,
        tuser.name1 as name1,
        tuser.birthday as birthday,
        tuser.sex as sex,
        tuser.created as created,
        tuser.updated as updated
        from
        tb_user tuser
        where tuser.id = #{id};
    </select>

    <select id="queryUserAll" resultType="com.springcloud.mybatis.pojo.User">
        select * from tb_user;
    </select>

    <!--插入数据-->
    <insert id="insertUser" parameterType="com.springcloud.mybatis.pojo.User">
        INSERT INTO tb_user (
        id,
        user_name,
        password,
        name1,
        age,
        sex,
        birthday,
        created,
        updated
        )
        VALUES
        (
        #{id},
        #{user_name},
        #{password},
        #{name1},
        #{age},
        #{sex},
        #{birthday},
        now(),
        now()
        );
    </insert>

    <update id="updateUser" parameterType="com.springcloud.mybatis.pojo.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=",">
            <if test="user_name!=null">user_name = #{user_name},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="name1!=null">name = #{name1},</if>
            <if test="age!=null">age = #{age},</if>
            <if test="sex!=null">sex = #{sex},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
            updated = now(),
        </trim>
        WHERE
        (id = #{id});
    </update>

    <delete id="deleteUser">
        delete from tb_user where id=#{id}
    </delete>
</mapper>
```

**Mapper层UserMapper.java**

需要与UserMapper.xml中的id对应!!

```java
package com.springcloud.produce.mapper;

import com.springcloud.produce.entity.User;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface UserMapper {
//根据id查询用户信息
    @Select("select * from tb_user where id = #{id}")
    public User queryUserById(String id);

    //查询所有用户信息
    @Select("select * from tb_user")
    public List<User queryUserAll();

    //新增用户
    @Insert("insert into tb_user(id,user_name, password,name1, age, sex, birthday, created, updated) values(#{id},#{user_name}, #{password},#{name1}, #{age}, #{sex}, #{birthday}, #{created}, #{updated})")
    public void insertUser(User user);

    //更新用户信息
    @Update("update tb_user set user_name = #{user_name},updated = #{updated} where id = #{id}")
    public void updateUser(User user);

    @Delete("delete from tb_user where id = #{id}")
    //根据id删除用户信息
    public void deleteUser(String id);
}
```

**service层：**

**UserService:**

```java
package com.springcloud.produce.service;

import com.springcloud.produce.entity.User;

import java.util.List;

public interface UserService {
    List<User getLists();

    public User queryUserById(String id);

    public void addUser(User user);

    public void deleteUser(String id);

    public void updateUser(User user);

}
```

**UserMapperImpl:**

```java
package com.springcloud.produce.service.impl;

import com.springcloud.produce.entity.User;
import com.springcloud.produce.mapper.UserMapper;
import com.springcloud.produce.service.UserService;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service
public class UserMapperImpl implements UserService{

    @Resource
    private UserMapper userMapper;
    
    @Override
    public List<User getLists() {
        List<User users = this.userMapper.queryUserAll();
        for(User user : users){
            System.out.println(user);
        }
        return users;
    }

    @Override
    public User queryUserById(String id) {
        return this.userMapper.queryUserById(id);
    }

    @Override
    public void addUser(User user) {
        this.userMapper.insertUser(user);
    }

    @Override
    public void deleteUser(String id) {
        this.userMapper.deleteUser(id);
    }

    @Override
    public void updateUser(User user) {
        this.userMapper.updateUser(user);
    }
}
```

**controller层：**

```java
package com.springcloud.produce.controller;

import com.springcloud.produce.entity.User;
import com.springcloud.produce.service.UserService;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import javax.sql.DataSource;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

@RestController
public class userController {

    @Autowired
    private UserService userService;

    @Resource
    private DataSource dataSource;

    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("YYYY-MM-dd HH:mm:ss");
    Date date = new Date();

    //这个很重要！！！构建sqlSessionFactory
    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean.getObject();
    }

    @RequestMapping("/selectUser")
    public User selectUser(){
        User user = this.userService.selectUser("1") ;
        return user;
    }

    @GetMapping("/{id}")
    public User queryUserById(@PathVariable String id){
        User user =this.userService.queryUserById(id);
        return user;
    }

    @RequestMapping("/")
    public String index(){
        return "hello";
    }

    @GetMapping("/list")
    public List<User lists(){
        List<User users = this.userService.getLists();
        return users;
    }

    @GetMapping("/addUser/{id}")
    public String addUser(@PathVariable String id) throws ParseException {
        User user = new User();
        user.setId(id);
        user.setAge(18);
        user.setSex(1);
        user.setBirthday(date);
        user.setCreated(simpleDateFormat.format(date));
        user.setUpdated(simpleDateFormat.format(date));
        user.setName1(id);
        user.setUser_name("wwww"+id);
        user.setPassword("123456");
        this.userService.addUser(user);
        return "success";
    }

    @GetMapping("/deteleUser/{id}")
    public String deleteUser(@PathVariable String id){
        this.userService.deleteUser(id);
        return "success";
    }

    @GetMapping("/updateUser/{id}")
    public String updateUser(@PathVariable String id){
        User user = this.userService.queryUserById(id);
        user.setUser_name("update");
        user.setUpdated(simpleDateFormat.format(date));
        this.userService.updateUser(user);
        return "success";
    }
}
```

最后运行项目测试一下，我 就不贴图了，测试过很多次了，如果有什么问题，欢迎留言~~

在消费模块中调用增删改查功能之后再写博客辣~~

[源码](https://download.csdn.net/download/www_indows/11818391)

下一篇打算总结一下mybatis的具体用法，总感觉自己还是没有弄的很清楚!!!