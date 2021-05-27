---
title: maven
tags:
  - maven
  - spring boot
categories:
  - spring boot
abbrlink: 21497
date: 2020-02-06 00:10:35
---

1. 在C:\Users\WHH\.m2下新建 settings.xml
<!-- more -->
2. 将以下内容加入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
    https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <mirror> 
            <id>alimaven</id> 
            <name>aliyun maven</name>                     
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>central</mirrorOf>         
        </mirror> 
    </mirrors>
</settings>
```

3. 重启IDEA