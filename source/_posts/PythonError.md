---
title: python Error
date: 2020-02-25 14:02:30
tags: python
categories: python
---

## 引言

记录python的各种错误！
<!-- more -->
## ModuleNotFoundError

- 关于ModuleNotFoundError: No module named 'urllib.request'; 'urllib' is not a package的问题

+ 重名导致urllib包无效化
  
  (1) 正在使用import urllib的文件命名就是urllib.py文件 
  
  (2) 正在使用的文件的文件夹目录下有一个urllib.py的文件，导致import urllib 实际上导入的是当前目录的urllib.py文件而不是urllib库文件

+ 解决方法：

    修改文件名即可

<!-- more -->

## Ubuntu18.04默认python2.7，安装python3.6并切换默认python版本

1.查看python版本

```python
python -V
# 安装python3.6
apt-get install python3.6
```

2.查看系统python的可用环境列表

```python
update-alternatives --list python
```

若显示 update-alternatives: error: no alternatives for python,则：

```python
# 版本号根据自己安装的设定
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives --install /usr/bin/python python /usr/bin/python3.6 2
```

3.配置python环境

```python
update-alternatives --config python
```

选择你想默认的版本的编号即可。

[参考文章](https://www.jb51.net/article/163117.htm)
