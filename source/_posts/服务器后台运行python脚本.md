---
title: 服务器后台运行python脚本
katex: false
date: 2020-10-09 16:46:25
updated: 2020-10-09 16:46:25
tags: python
categories: python
---

服务器运行python

```bash
python xx.py
```
<!-- more -->
后台运行python

& 符号 代表 后台运行，如果用户登出后，脚本也就自动退出了

```bash
python xx.py &
```

nohup 保证了用户登出之后，脚本还会继续后台运行

```bash
nohup python xxx.py &
```

```bash
nohup python xx.py > /PATH/run.log 2>&1 &
```
0 – stdin (standard input)，1 – stdout (standard output)，2 – stderr (standard error) 

2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到run.log文件中。