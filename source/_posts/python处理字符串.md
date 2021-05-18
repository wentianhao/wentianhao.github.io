---
title: python处理字符串
date: 2020-02-22 14:44:36
tags: python
categories: python 
---

1. 使用正则表达式来获取字符串中需要的字符

```python
re.compile(正则表达式).findall(源字符串)
```

[正则表达式基础](https://whh.plus/2020/02/19/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%9F%BA%E7%A1%80/)
<!-- more -->
2. strip(),rstrip(),lstrip()

```python
str = '  ---abc123+++  '
# 删除两边空字符
print(str.strip())
# 删除左边空字符
print(str.lstrip())
# 删除右边空字符
print(str.rstrip())
# 删除两边的 - + 和空字符
print(str.strip().strip('-+'))
```

3. 删除单个固定位置字符 : 切片 + 拼接

```python
s = 'abc:123'
#字符串拼接方式去掉冒号
s1 = s[:3]+s[4:]
print(s1)
```

4. 删除任意位置字符同时删除多种不同字符：replace(),re.sub()

```python
# 去掉字符串中相同的字符
s = 'af123adsa'
print(s.repace('a',''))
```

```python
import re
s = '\r\nabc\t123\nxyz'
# 去除\r \n \t 字符
print(re.sub('[\r\n\t]','',s))
```

5. 同时删除多种不同字符 translate() str.maketrans()做映射

```python
s='abc123xyz'
# a _> x,b_>y,c_>z，字符映射加密
print(str.maketrans('abcxyz','xyzabc'))
# translate把其转换成字符串
print(s.translate(str.maketrans('abcxyz','xyzabc')))
```

6. 分割字符串

- str.split()

利用字符串函数split分割，分割后返回的是分割部分的字符列表，每次只能处理一个分隔符

```python
str='abc,123 efg,567'
str.split()
# print : ['abc,123','efg,567']
str.split(',')
# print : ['abc','123 efg','567']
```

- re.split()

利用re模块的split函数对字符串进行分割，分割后返回的是分割部分的字符列表，可以实现一次性分割多种分隔符

```python
import re
str='abc,123 efg,567'
re.split('[, ]',str)
# print : ['abc','123','efg','567']
```

- str.partition()
- 
利用字符串函数partition或者rpartition实现字符串分割

这俩函数都只接收一个分割字符串作为参数,将目标字符串分割为两个部分，返回一个三元元组(head,sep,tail),包含分割符。细微区别在于前者从目标字符串的末尾也就是右边开始搜索分割符

```python
str = "abc,123 efg,567"
str.partition(',')
# print:('abc',',','123 efg,567')
str.rpartition(',')
# print: ('abc,123 efg',',','567')
```
