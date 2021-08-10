---
title: argparse基本用法
katex: false
tags: python
categories: python
abbrlink: 30536
date: 2020-10-21 11:20:51
updated: 2020-10-21 11:20:51
---

argparse 是 python自带的命令行参数解析包，用来方便读取命令行参数

<!-- more -->

## 用法

- 导入argparse包
- 利用ArgumentParser类生成parser对象，description可为空
- 通过add_argument函数新增参数。'-n'和'--name'表示同一参数，'default'表示运行时若没有提供参数，则当默认值
- parse_args获取解析的参数

```python
import argparse

parser = argparse.ArgumentParser(description='what')

parser.add_argument('-n','--name',default='WEN')
parser.add_argument('-y','--year',default='20')
args = parser.parse_args()
print(args)
name  = args.name
year = args.year
print('Hello {} {}'.format(name,year))
```

运行`python test.py`

结果
```bash
Namespace(name='WEN', year='20')
Hello WEN 20
```

运行`python test.py -n LI -y 31`

结果
```bash
Namespace(name='LI', year='31')
Hello LI 31
```