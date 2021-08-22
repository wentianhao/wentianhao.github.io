---
title: SpringBoot-表单验证-统一异常处理-自定义验证信息源
katex: false
tags: spring boot
categories: spring boot
abbrlink: 17240
date: 2021-08-20 21:29:16
---
前端验证一般是为了满足界面的友好性、客户体验性等，但仅靠前端进行数据合法性校验，是不够的。因为非法用户可能会直接从客户端获取请求地址进行非法请求，所以后端的校验是必不可少的。

在书写业务逻辑时，经常有大量的判空校验。比如Service层或Dao层的方法入参、入参对象、出参等都有一套校验规则。比如有的字段必传，有的非必传，返回值有些字段必须有值，有的非必须等等

众多if else的重复无意义代码，也可称为垃圾代码。Bean Validation校验是基于DDD思想设计的。

## 项目结构
![项目结构图](https://whh.plus/images/valid1.png)
```bash
├── java
│   └── com
│       └── whh
│           └── valid
│               ├── ValidApplication.java # 启动类
│               ├── annotation
│               │   └── Phone.java # 自定义验证注解
│               ├── config
│               │   └── ValidatorConfig.java # 表单验证配置类
│               ├── controller
│               │   └── SysUserController.java # 用户管理控制器
│               ├── exception
│               │   ├── BusinessException.java # 业务异常类
│               │   └── GlobalExceptionHandler.java # 统一异常处理类
│               ├── entity
│               │   ├── SysUser.java # 用户信息实体
│               │   └── ValidationInterface.java # 表单验证的通用分组接口
│               ├── util
│               │   └── CommonResult.java # 接口返回封装类
│               └── validation
│                   └── PhoneValidator.java #自定义验证实现类
└── resources
    ├── application.yaml # 配置文件
    └── messages
        └── validation
            └── messages.properties # 自定义验证信息源
```
### 导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.whh</groupId>
    <artifactId>valid</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>valid</name>
    <description>表单验证demo</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!-- web支持 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 表单验证 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.1.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.3.5.Final</version>
        </dependency>
</dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 添加实体类
```java
package com.whh.valid.enitty;

import lombok.Data;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Size;

@Data
public class SysUser {
    private static final long serialVersionUID = 1L;

    // 主键
    private Long id;

    //用户名
    @NotEmpty(message = "用户名不能为空")
    private String username;

    // 密码
    @Size(min = 6,max = 16,message = "密码长度必须在{min} - {max}之间")
    private String password = "123456";

    // 邮箱地址
    @Email(message = "邮箱地址不合法")
    @NotEmpty(message = "邮箱不能为空")
    private String email;

    // 电话
    @Size(min = 11,max = 11,message = "手机号不合法")
    @NotEmpty(message = "手机号不能为空")
    private String phone;
}
```
@Data注解：提高代码的简洁，使用这个注解可以省去代码中大量的get()、 set()、 toString()等方法；

### 接口返回封装类
```java
package com.whh.valid.util;

import lombok.Data;
import lombok.NoArgsConstructor;


// 操作消息提醒
@Data
@NoArgsConstructor
public class CommonResult {
    // 状态码
    private int code;

    // 返回内容
    private String msg;

    // 数据对象
    private Object data;

    // 初始化一个CommonResult对象
    public CommonResult(Type type,String msg){
        this.code = type.value;
        this.msg = msg;
    }

    public CommonResult(Type type,String msg,Object data){
        this.code = type.value;
        this.msg = msg;
        if (data != null){
            this.data = data;
        }
    }

    // 返回成功消息
    public  static CommonResult success() {
        return CommonResult.success("操作成功");
    }

    public  static CommonResult success(Object data) {
        return CommonResult.success("操作成功",data);
    }

    public static CommonResult success(String msg) {
        return CommonResult.success(msg,null);
    }

    public static CommonResult success(String msg,Object data) {
        return new CommonResult(Type.SUCCESS,msg,data);
    }

    // 返回警告消息
    public static CommonResult warn(String msg) {
        return CommonResult.warn(msg,null);
    }
    public static CommonResult warn(String msg,Object data) {
        return new CommonResult(Type.WARN,msg,data);
    }

    // 返回错误消息
    public static CommonResult error(){
        return CommonResult.error("操作失败");
    }

    public static CommonResult error(String msg) {
        return CommonResult.error(msg,null);
    }

    public static CommonResult error(String msg,Object data) {
        return new CommonResult(Type.ERROR,msg,data);
    }

        public enum Type{
        // 成功
        SUCCESS(200),
        // 警告
        WARN(301),
        // 错误
        ERROR(500);
        private final int value;

        Type(int value){
            this.value = value;
        }

        public int getValue(){
            return this.value;
        }
    }
}
```
@NoArgsConstructor注解：注解在类上，为类提供一个无参的构造方法。

### 控制器
```java
package com.whh.valid.controller;

import com.whh.valid.enitty.SysUser;
import com.whh.valid.util.CommonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

@Slf4j
@RestController
@RequestMapping("sys/user")
public class SysUserController {
    private static final List<SysUser> USERS = new ArrayList<>();

    // 数据初始化
    static {
        SysUser user = new SysUser();
        user.setId(1L);
        user.setUsername("wen");
        user.setPhone("13666666666");
        user.setEmail("example@qq.com");
        USERS.add(user);
        SysUser user1 = new SysUser();
        user1.setId(2L);
        user1.setUsername("li");
        user1.setPhone("13688888888");
        user1.setEmail("example1@qq.com");
        USERS.add(user1);
    }

    // 新增用户
    @PostMapping
    public CommonResult add(@Validated @RequestBody SysUser sysUser, BindingResult result) {
        FieldError fieldError = result.getFieldError();

        if (Objects.nonNull(fieldError)) {
            String field = fieldError.getField();
            Object rejectedValue = fieldError.getRejectedValue();
            String msg = "[" + fieldError.getDefaultMessage() + "]";
            log.error("{}: 字段=={}\t值={}",msg,field,rejectedValue);
            return CommonResult.error(msg);
        }

        USERS.add(sysUser);
        return CommonResult.success("新增成功");
    }
}
```
- @Validated注解：标识需要验证的类，使用BindingResult类接受错误信息
- @Slf4j注解：用作日志输出的，一般会在项目每个类的开头加入该注解，如果不写下面这段代码，并且想用log
```private final Logger logger = LoggerFactory.getLogger(当前类名.class);```

### 测试
![测试](https://whh.plus/images/test1.png)
log
`[邮箱地址不合法]: 字段==email	值=123`

## 分组校验
groups表示注解属于哪个组，便按照哪个组的校验规则进行校验

当在controller中校验表单数据时，如果使用了groups，那么没有在这个分组下的属性是不会校验的

### 添加分组接口
```java
package com.whh.valid.enitty;

// 用于表单验证的通用分组接口
public interface ValidationInterface {
    /**
     *新增分组
     */
    interface add{}

     /**
      * 删除分组
      */
    interface delete{}

    /**
     *查询分组
     */
    interface select{}

    /**
     * 更新分组
     */
    interface update{}
}
```
如果还有其它特殊的分组要求 直接在DO中创建interface即可

### 修改实体类
将属性进行分组
```java
package com.whh.valid.enitty;

import lombok.Data;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

@Data
public class SysUser {
    private static final long serialVersionUID = 1L;

    // 主键
    @NotNull(message = "id不能为空",groups = {ValidationInterface.update.class})
    private Long id;

    //用户名
    @NotEmpty(message = "用户名不能为空",groups = {
            ValidationInterface.update.class,
            ValidationInterface.add.class
    })
    private String username;

    // 密码
    @Size(min = 6,max = 16,message = "密码长度必须在{min} - {max}之间",groups = {
            ValidationInterface.update.class,
            ValidationInterface.add.class
    })
    private String password = "123456";

    // 邮箱地址
    @Email(message = "邮箱地址不合法",groups = {
            ValidationInterface.update.class,
            ValidationInterface.add.class,
            ValidationInterface.select.class
    })
    @NotEmpty(message = "邮箱不能为空",groups = ValidationInterface.add.class)
    private String email;

    // 电话
    @Size(min = 11,max = 11,message = "手机号不合法",groups = {
            ValidationInterface.update.class,
            ValidationInterface.add.class,
            ValidationInterface.select.class
    })
    @NotEmpty(message = "手机号不能为空",groups = {ValidationInterface.add.class})
    private String phone;
}
```

### 修改控制器
添加操作方法，并且方法形参上指定验证的分组
```java
package com.whh.valid.controller;

import com.whh.valid.enitty.SysUser;
import com.whh.valid.enitty.ValidationInterface;
import com.whh.valid.util.CommonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

@Slf4j
@RestController
@RequestMapping("sys/user")
public class SysUserController {
    private static final List<SysUser> USERS = new ArrayList<>();

    // 数据初始化
    static {
        SysUser user = new SysUser();
        user.setId(1L);
        user.setUsername("wen");
        user.setPhone("13666666666");
        user.setEmail("example@qq.com");
        USERS.add(user);
        SysUser user1 = new SysUser();
        user1.setId(2L);
        user1.setUsername("li");
        user1.setPhone("13688888888");
        user1.setEmail("example1@qq.com");
        USERS.add(user1);
    }

    /**
     * 根据手机号或邮箱查询用户信息
     * @param sysUser 用户信息
     * @param result 错误反馈
     * @return 用户信息
     */
    @GetMapping
    public CommonResult queryList(@Validated(value = ValidationInterface.select.class) SysUser sysUser,BindingResult result) {
        FieldError fieldError = result.getFieldError();

        if (Objects.nonNull(fieldError)) {
            return CommonResult.error(getErrorMsg(fieldError));
        }

        String phone = sysUser.getPhone();
        String email = sysUser.getEmail();

        if (phone == null && email == null){
            return CommonResult.success(USERS);
        }

        List<SysUser> queryResult = USERS.stream()
                .filter(obj -> obj.getPhone().equals(phone) || obj.getEmail().equals(email))
                .collect(Collectors.toList());
        return CommonResult.success(queryResult);
    }

    /**
     * 获取表单验证错误msg
     * @param fieldError 报错字段
     * @return msg
     */
    public String getErrorMsg(FieldError fieldError) {
        String field = fieldError.getField();
        Object rejectedValue = fieldError.getRejectedValue();
        String msg = "[" + fieldError.getDefaultMessage() + "]";
        log.error("{}: 字段=={}\t值=={}",msg,field,rejectedValue);
        return msg;
    }

    /**
     * 新增用户信息
     * @param sysUser 用户信息
     * @param result 错误反馈
     * @return 成功标识
     */
    @PostMapping
    public CommonResult add(@Validated(value = ValidationInterface.add.class)
                                @RequestBody SysUser sysUser,
                                BindingResult result) {
        FieldError fieldError = result.getFieldError();

        if (Objects.nonNull(fieldError)) {
            return CommonResult.error(getErrorMsg(fieldError));
        }
        Long id = (long)(USERS.size() + 1);
        sysUser.setId(id);
        USERS.add(sysUser);
        return CommonResult.success("新增成功");
    }

    /**
     * 根据id更新用户信息
     * @param id id
     * @param sysUser 用户信息
     * @param result 错误反馈
     * @return 成功标识
     */
    @PutMapping("{id}")
    public CommonResult updateByid(@PathVariable("id") Long id,
                                    @Validated(value = ValidationInterface.update.class)
                                    @RequestBody SysUser sysUser,
                                    BindingResult result) {
        FieldError fieldError = result.getFieldError();

        if (Objects.nonNull(fieldError)) {
            return CommonResult.error(getErrorMsg(fieldError));
        }

        for (int i=0;i<USERS.size();i++){
            if (USERS.get(i).getId().equals(id)) {
                USERS.set(i,sysUser);
            }
        }
        return CommonResult.success("更新成功");
    }

    /**
     * 根据Id删除用户信息
     * @param id id
     * @return 成功标识
     */
    @DeleteMapping("{id}")
    public CommonResult deleteByid(@PathVariable Long id) {
        USERS.removeIf(obj -> obj.getId().equals(id));
        return CommonResult.success("删除成功");
    }
}
```

### 测试
**查询**：输入不合法手机号
![get](https://whh.plus/images/get.png)

**新增**
![add](https://whh.plus/images/add.png)
去掉邮箱
![add1](https://whh.plus/images/add1.png)

**更新**：去掉id
![put](https://whh.plus/images/put.png)

**删除**
![delete](https://whh.plus/images/delete.png)

## 自定义验证
很多时候框架提供的功能并不能满足业务场景，需要自定义一些验证规则来完成验证。

### 添加注解
```java
package com.whh.valid.annotation;


import com.whh.valid.validation.PhoneValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Documented
@Constraint(validatedBy = {PhoneValidator.class})
@Target({METHOD,FIELD,ANNOTATION_TYPE,CONSTRUCTOR,PARAMETER,TYPE_USE})
@Retention(RUNTIME)
public @interface Phone {
    // 默认错误消息
    String message() default "不是一个合法的手机号";

    // 分组
    Class<?>[] groups() default {};

    // 载荷 将某些 元数据信息与给定的注解声明相关联的方法
    Class<? extends Payload>[] payload() default {};

    //指定多个时使用
    @Target({FIELD,METHOD,PARAMETER,ANNOTATION_TYPE})
    @Retention(RUNTIME)
    @Documented
    @interface List{
        Phone[] value();
    }
}
```
- @Documented：是否将注解信息添加在Java文档中
- @Retention：定义该注解的生命周期.(SOURCE:在编译阶段丢弃。这些注解在编译结束之后就不再有意义。例如@Override，@SuppressWarnings;CLASS：在类加载阶段丢弃，在字节码文件的处理中有用。注解默认使用这种方式；RUNTIME：始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解信息，自定义注解通常使用这种方式)
- @Target：注解用于什么地方，如果不明确指出，该注解可以放在任何地方。属性的注解是兼容的。(TYPE：用于描述类、接口或enum声明；FIELD：用于描述实例变量；METHOD：用于描述方法；PARAMETER：参数；CONSTRUCTOR：构造方法；LOCAL_VARIABLE：局部变量；ANNOTATION_TYPE：另一个注释；PACKAGE：用于记录java文件的package信息)
- @Constraint：处理注解的逻辑

### 编写验证逻辑
```java
package com.whh.valid.validation;

import com.whh.valid.annotation.Phone;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.Objects;
import java.util.regex.Pattern;

/**
 * 手机号校验器
 */
public class PhoneValidator implements ConstraintValidator<Phone,String> {
    /**
     * 手机号正则表达式
     */
    private static final String REGEXP_PHONE = "^1[3456789]\\d{9}$";

    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        if (Objects.isNull(s)){
            return true;
        }
        return Pattern.matches(REGEXP_PHONE,s);
    }
}
```
### 修改实体
给phone添加注解@Phone
```java
@Phone(groups = {
                ValidationInterface.update.class,
                ValidationInterface.add.class,
                ValidationInterface.select.class})
@NotEmpty(message = "手机号不能为空", groups = {ValidationInterface.add.class})
private String phone;
```

### 测试
输入错误的手机号测试
![phone](https://whh.plus/images/phone.png)

### @Pattern
validation也提供了基于正则匹配的注解@Pattern
```java
@Pattern(message = "手机号不合法",regexp = "^1[3456789]\\d{9}"$,groups = {ValidationInterface.add.class})
@NotEmpty(message = "手机号不能为空", groups = {ValidationInterface.add.class})
private String phone;
```

## 调用过程验证
有时候在参数传输过程中需要对传入的对象做参数验证，但是上面介绍的是对参数绑定时的验证。
### 使用 spring bean
- 注入validator
bean validator是在config中定义的bean，如果使用spring boot默认配置ValidationAutoConfiguration::defaultValidator(),直接注入bean
```java
@Resource(name = "validator")
javax.validation.Validator validator;
```
- 

## 方法参数验证
- 修改控制器
直接在类上添加注解@Validated，并在方法上直接进行验证
```java
@Slf4j
@Validated
@RestController
@RequestMapping("sys/user")
public class SysUserController {
  ... 省略代码
    /**
     * 根据手机号和邮箱查询用户信息
     * @param phone 手机号
     * @return 用户list
     */
    @GetMapping("selectByPhone")
    public CommonResult queryByPhone(@NotEmpty(message = "手机号不能为空") String phone) {
       List<SysUser> queryResult = USERS.stream()
             .filter(obj -> obj.getPhone().equals(phone))
             .collect(Collectors.toList());
       return CommonResult.success(queryResult);
    }
}
```
- 测试
![get](https://whh.plus/images/get1.png)

## 统一异常处理
上面参数验证时，验证的错误信息是通过`BindingResult result`参数进行接收的，比较麻烦。甚至是直接将异常堆栈信息返回给前端，不太友好，有的时候需要主动抛出业务异常，比如用户不能直接删除已绑定用户的角色。
### 创建业务异常类
```java
package com.whh.valid.exception;

// 业务异常
public class BusinessException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    protected final String message;


    public BusinessException(String message) {
        this.message = message;
    }

    public BusinessException(String message,Throwable e) {
        super(message,e);
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }
}
```
### 创建全局异常处理器
```java
package com.whh.valid.exception;

import com.whh.valid.util.CommonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import javax.servlet.http.HttpServletRequest;
import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import javax.validation.ValidationException;
import java.util.Set;

/**
 * 全局异常处理器
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * 参数绑定异常类，用于表单验证时抛出异常处理
     */
    @ExceptionHandler(BindException.class)
    public CommonResult validatedBindException(BindException e) {
        log.error(e.getMessage(),e);
        BindingResult bindingResult = e.getBindingResult();
        FieldError fieldError = e.getFieldError();
        String message = "[" + e.getAllErrors().get(0).getDefaultMessage()+"]";
        return CommonResult.error(message);
    }

    /**
     * 用于方法形参中参数校验时抛出的异常处理
     * @param e
     * @return
     */
    @ExceptionHandler(ConstraintViolationException.class)
    public CommonResult handle(ValidationException e) {
        log.error(e.getMessage(),e);
        String errorInfo = "";
        if (e instanceof ConstraintViolationException) {
            ConstraintViolationException exs = (ConstraintViolationException) e;
            Set<ConstraintViolation<?>> violations = exs.getConstraintViolations();
            for (ConstraintViolation<?> item : violations){
                errorInfo = errorInfo + "[" + item.getMessage() + "]";
            }
        }
        return CommonResult.error(errorInfo);
    }

    /**
     * 请求方式不支持
     * @param e
     * @return
     */
    @ExceptionHandler({HttpRequestMethodNotSupportedException.class})
    public CommonResult handleException(HttpRequestMethodNotSupportedException e){
        log.error(e.getMessage(),e);
        return CommonResult.error("不支持" + e.getMethod()+"请求");
    }

    /**
     * 拦截未知的运行时异常
     * @param e
     * @return
     */
    @ExceptionHandler(RuntimeException.class)
    public CommonResult notFount(RuntimeException e){
        log.error("运行时异常",e);
        return CommonResult.error("运行时异常",e.getMessage());
    }

    /**
     * 系统异常
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    public CommonResult handleException(Exception e){
        log.error(e.getMessage(),e);
        return CommonResult.error("服务器错误，请联系管理员");
    }

    /**
     * 业务异常
     * @param request
     * @param e
     * @return
     */
    @ExceptionHandler(BusinessException.class)
    public CommonResult bussinessException(HttpServletRequest request,BusinessException e){
        log.error(e.getMessage());
        return CommonResult.error(e.getMessage());
    }
}
```
- @Slf4j注解：用作日志输出的，一般会在项目每个类的开头加入该注解，如果不写下面这段代码，并且想用log
- @RestControllerAdvice：对Controller增强，捕获全局Spring MVC抛出的异常。
- @ExceptionHandler(BindException.class)：用来捕获指定的异常

### 修改控制器
删除方法中的BindingResult result参数，将错误直接抛给统一异常处理类去解决即可

### 测试
![test](https://whh.plus/images/testException.png)

## 自定义验证信息源
### 修改配置文件(ValidatorConfig)
```java
/**
     * 实体类字段校验国际化引入
     * */

    @Bean
    public Validator validator() {
//        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
//                .configure()
//                .addProperty("hibernate.validator.fail_fast","true")
//                .buildValidatorFactory();
        LocalValidatorFactoryBean validator = new LocalValidatorFactoryBean();
        // 设置messages资源信息
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        // 多个用逗号分割
        messageSource.setBasenames("classpath:/messages/validation/messages");
        // 设置字符集编码
        messageSource.setDefaultEncoding("UTF-8");
        validator.setValidationMessageSource(messageSource);
        // 设置验证相关参数
        Properties properties = new Properties();
        // 快速失败，有错即返回
        properties.setProperty("hibernate.validator.fail_fast", "true");
        validator.setValidationProperties(properties);
//        return validatorFactory.getValidator();
        return validator;
    }
```
### 添加信息源文件
```bash
├───resources
    └── messages
        └── validation
            └── messages.properties
```

```properties
# messages.properties
name.not.empty=用户名不能为空
email.not.valid=${validatedValue}是邮箱地址？
email.not.empty=邮箱不能为空
phone.not.valid=${validatedValue}是手机号？
phone.not.empty=手机号不能为空
password.size.valid=密码长度必须在{min}-{max}之间
id.not.empty=主键不能为空
```

### 修改实体类
```java
message = "{name.not.empty}"
message = "{password.size.valid}"
message = "{email.not.valid}"
message = "{phone.not.valid}"
```

## 注解
|注解 | 说明| 
|:---: | :---: |
| @Null | 限制只能为null |
| @NotNull | 限制必须不为null
| @AssertFalse | 限制必须为false|
| @AssertTrue | 限制必须为true|
| @DecimalMax(value) | 限制必须为一个不大于指定值的数字|
| @DecimalMin(value) | 限制必须为一个不小于指定值的数字|
| @Digits(integer,fraction) | 限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction|
| @Future | 限制必须是一个将来的日期 |
| @Max(value) | 限制必须为一个不大于指定值的数字|
| @Min(value) | 限制必须为一个不小于指定值的数字|
| @Past | 限制必须是一个过去的日期|
| @Pattern(value) | 限制必须符合指定的正则表达式|
| @Size(max,min) | 限制字符长度必须在min到max之间|
| @Past | 验证注解的元素值（日期类型）比当前时间早|
| @NotEmpty | 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）|
| @NotBlank | 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank|
| @Email | 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式|


## 参考
[开撸！SpringBoot-表单验证-统一异常处理-自定义验证信息源](https://www.toutiao.com/i6994693398586393096/)
[Spring方法级别数据校验](https://blog.csdn.net/f641385712/article/details/97402946)
[@Data注解 与 lombok](https://www.jianshu.com/p/c1ee7e4247bf)
[SpringBoot - Lombok使用详解3（@NoArgsConstructor、@AllArgsConstructor、@RequiredArgsConstructor）](https://www.hangge.com/blog/cache/detail_2493.html)
[@Constraint注解配合自定义验证类型注解的开发](https://blog.csdn.net/lwg_1540652358/article/details/84193759)
[Spring Boot @RestControllerAdvice 统一异常处理](https://www.jianshu.com/p/47aeeba6414c)