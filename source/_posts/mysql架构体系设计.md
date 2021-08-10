---
title: mysql架构体系设计
katex: false
tags: mysql
categories: mysql
abbrlink: 60458
date: 2021-06-22 12:22:55
---

## Mysql 记录存储
![mysql](https://whh.plus/images/mysql.png)
- 页头
- 虚记录
- 记录堆
- 自由空间链表
- 未分配空间
- Slot区
- 页尾
