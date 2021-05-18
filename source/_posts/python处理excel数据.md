---
title: python处理excel数据
date: 2020-02-22 13:17:07
tags: python
categories: python
---

1. 安装xlrd 和xlwt 库(python3)

```python
pip install xlrd,xlwt
```
<!-- more -->
2.使用方法

```python
import xlrd,xlwt
from xlutils.copy import copy

# 打开需要操作的excel表
excel1 = xlrd.open_workbook(path)

# 获取excel的sheet表
sheet1 = excel1.sheet_by_name(sheet_name)

# rows & cols

rows = sheet1.nrows
cols = sheet1.ncols

# 第i行、列的数据
row_i = sheet1.row_values(i)
col_i = sheet1.col_values(i)

# 查看所有的sheet表
sheet_names = excel1.sheet_names()

# copy excel1 
excel2 = copy(excel1)

# 新建表
excel3 = xlwt.Workbook(encoding = 'utf-8')

# 创建sheet
sheet_name = excel2.add_sheet(name)

#获取该sheet的最后一行
n = len(sheet_name.rows)

# 写入excel ,row col value
sheet_name.write(row,col,value)

#在原有sheet后新增数据
sheet_name.write(n,col,value)

# 保存 生成excel表
excel2.save(path)
```
