---
title: linux命令：useradd
katex: false
tags: linux
categories: linux
abbrlink: 32725
date: 2021-03-29 21:21:51
---

## 每天一个Linux命令：useradd

ubuntu创建新用户并赋予指定目录的相关权限

<!-- more -->

```bash
useradd username   # 添加用户 username
passwd username   # 输入密码
usermod -s /bin/bash username  # 为该用户指定命令解释程序
usermod -d /home/username  # 为该用户指定用户主目录
cat /etc/passwd   # 查看用户属性
```

赋予用户主目录下的读写权限
```bash
chown -R username:username /home/username
chmod 755 /home/username
```

赋予用户root权限
```bash
visudo  # 打开 /etc/sudoers文件
username ALL =(ALL:ALL) ALL # root下加入
```


参考文献：
[https://www.jb51.net/article/175195.htm](https://www.jb51.net/article/175195.htm)
[https://blog.csdn.net/jiangjiang_jian/article/details/81169258](https://blog.csdn.net/jiangjiang_jian/article/details/81169258)