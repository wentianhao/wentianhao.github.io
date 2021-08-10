---
title: json
katex: false
tags: json
categories: json
abbrlink: 8000
date: 2021-06-28 13:55:21
---


JSON必知必会
<!-- more -->

##  什么是json
json是一种轻量的数据交换格式

### json是一种数据交换格式
数据交换格式是一种在不同平台间传递数据的文本格式。JSON、XML

并不是所有的系统都支持使用JSON来交换数据。许多系统还在使用XML这样的格式，或是用表格和分隔符来表示数据的格式，如逗号分隔值(CSV)

### JSON独立于编程语言
JSON(JavaScript Object Notation, JavaScript对象表示法)

JSON基于JavaScript对象字面量，一种基于对象表示法的数据交换格式

### 专业术语和概念
- JSON ： JavaScript对象表示法
- 表示法： 一个用于表示诸如数字或单词等数据的字符系统
- 数据交换格式： 用于在不同的平台或系统间交换数据的文本
- 可移植性： 以一种对双方系统都兼容的方式在平台间传递信息

## JSON语法

### JSON基于JavaScript对象字面量
字面量：数据值的具体表示

例如用一些特征或者属性，比如颜色、风格、品牌等来描述你的鞋子，名称-值对
```json
{
    "brand": "Crocs",
    "color": "pink",
    "size": 9,
    "hasLaces": false
}
```

### 名称-值对
名称始终在左侧，值始终在右侧

### 正确的JSON语法
JSON中的名称-值对的名称如果被系统作为对象装入内存的话，将会成为“属性”。在部分系统中，属性名可以包含下划线(_)或数字，但是大多数情况下最好使用英文字母A-Z或a-z。

当值为字符串时，必须使用双引号，为其他数据类型时，不需要双引号，在JSON中，我们仅使用双引号，而且对于名称-值中的名称来说，是必需的

### 语法验证
语法验证关注的是JSON格式，而一致性验证关注的是其独特的数据结构，语法验证会检测JSON语法是否正确(是否被花括号包裹，名称-值对是否以逗号分隔)，而一致性验证会检测数据中是否包含name、breed和age等信息，还会检测age的值是否为数字，name的值是否为字符串等

### 媒体类型
JSON的MIME类型是 application/json

### 专业术语和概念
- 字面量：字面意思与其想要表达的意思是完全一致的值
- 变量：通过形如x的标识符来表示的，可以修改的一类值
- 最大可移植性(数据交换中)：通过保证数据本身对于平台和系统的兼容性来提供超越数据格式本身的可移植性
- 名称-值对：拥有名称和对应的值的属性或特征(也叫键-值对)
- 语法验证：关注JSON格式的验证
- 一致性验证：关注独特数据结构的验证

## JSON的数据类型
### 数据类型简介
原始数据类型：
- 数字(5或5.09)整型、浮点数、定点数
- 字符和字符串(如"a"、"A"或"apple")
- 布尔类型(即真或假)

### JSON中的数据类型
JSON中的数据类型包括：
- 对象
- 字符串
- 数字
- 布尔值
- null
- 数组

### JSON中的对象类型
JSON本身就是一个对象.
```json
{
    "person":{
    "name": "wanheo",
    "height": 66,
    "head": {
        "hair":{
            "color": "light blond",
            "length": "short"
            },
        "eyes": "green"
        }
    }
}
```

### JSON中的字符串类型
JSON中的字符串可以有任何UniCode字符构成，字符串两边必须用双引号包裹

错误代码,不能正常使用
```json
{
    "promo": "Say "Bob's the best!" at checkout for free 80z bag of kibble."
}
```

使用反斜杠对字符串中的双引号进行转义来解决这一问题
```json
{
    "promo": "Say \"Bob's the best!\" at checkout for free 80z bag of kibble."
}
```
反斜线会告诉解析器这个双引号并不意味着字符串的结束。双引号并不是唯一需要转义的字符，因为反斜线用于转义字符，所以还需要转义反斜线

错误代码
```json
{
    "location": "C:\Program Files"
}
```
反斜线需要另一个反斜线进行转义
```json
{
    "location": "C:\\Program Files"
}
```

除了双引号和反斜线，还需要转义以下字符：
- \/(正斜线)
- \b(退格符)
- \f(换页符)
- \t(制表符)
- \n(换行符)
- \r(回车符)
- \u后面跟十六进制字符(如笑脸\u263A)

### JSON中的数字类型
JSON中的数字可以是整数、小数、负数或指数
```json
{
    "widgeInventory": 289,
    "sadSavingsAccount": 22.59,
    "seattleLatitude": 47.606209,
    "earthsMass": 5.97219e+24
}
```

### JSON中的布尔类型
在JSON中，该字面仅使用小写形式：true或false
```json
{
    "toastWithBreakfast": false,
    "breadwithLunch": true
}
```

### JSON中的null类型
null用来表示0、一无所有、不存在等意思，而不用数字表示。

undefined不是JSON中的数据类型，在JavaScript中，undefined是在尝试获取一些不存在的对象或变量时返回的结果，undefined与那些声明的名称和值都不存在的对象或变量有关，而null则仅与对象或变量的值有关

在JSON中，null必须使用小写形式

### JSON中的数组类型
```json
{
    "eggCartom": [
        "egg",
        null,
        1
    ]
}
```

```json
{
    "test":[
        {
            "question": "The sky is blue",
            "answer": true
        },
        {
            "question": "The earth is flat",
            "answer": false
        }
    ]
}
```

### 专业术语和概念
- JSON中的字符串类型：一个字符串值，如"string",使用双引号包裹
- JSON中的布尔类型：true 或 false
- JSON中的数字类型：一个数字值，如42，-20，1.5，3e+10
- JSON中的null类型：一个表示空值的null
- JSON中的数组类型：数组是值的集合或列表，每个值可以是字符串、数字、布尔值、对象或数组中的任何一种，数组必须用方括号([])包裹，值于值之间用逗号隔开
- JSON中的对象类型：对象类型是使用逗号分隔的名称-值对构成的集合，并使用花括号({})包裹

对象和数组很关键的一个区别是： 对象是名称-值对构成的列表或集合，数组是值构成的列表或集合

对象和数组另一个关键的区别是：数组中所有的值应具有相同的数据类型

## JSON Schema
```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Cat",
    "properties": {
        "name": {
            "type": "string"
        },
        "name": {
            "type": "number",
            "description": "Your cat's age in years"
        },
        "declawed": {
            "type": "boolean"
        }
    },
    "required": [
        "name",
        "age",
        "declawed"
    ]
}
```

### 专业术语和概念
- JSON Schema：数据交换中的一种虚拟的“合同”
- JSON验证器负责验证语法错误，JSON Schema负责提供一致性检验
- JSON Schema 是数据接收方的第一道防线，也是数据发送方节约时间、保证数据正确的好工具

## JSON中的安全问题
GET 请求数据，得到相应

POST 提交数据，得到相应 

----------------------------
## **小结**

JSON优点：
- JSON语法小轻量级，以更快的方式进行相应
- 兼容性广泛
- 共享数据的最佳工具
- 在服务器上解析

缺点：
- 没有错误处理
- 完全性，JSON服务返回包含在函数调用中的JSON响应，如果它将与不受信任的浏览器一起使用，将由浏览器执行它可以被黑客入侵，这使托管Web应用程序易受各种攻击
- 支持工具有限