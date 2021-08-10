---
title: Urllib爬虫项目
tags:
  - python
  - python爬虫
categories: python
abbrlink: 51632
date: 2020-02-25 12:51:36
---
Urllib模块是一个可以用于编写爬虫的非常常用的模块
<!-- more -->

```python
import urllib
import urllib.request
import re
import random

url = "http://www.jd.com"

# 爬到内存中
jd = urllib.request.urlopen(url).read().decode("utf-8","ignore")
print(len(jd))
pat = "<title>(.*?)</title>"
title = re.compile(pat,re.S).findall(jd)
print(title)

# 爬到硬盘文件中
urllib.request.urlretrieve(url,filename="./jd.html")

# 浏览器伪装
url2 = "https://www.qiushibaike.com/"
opener = urllib.request.build_opener()
UA = ("User-Agent",
"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36")
opener.addheaders=[UA]
urllib.request.install_opener(opener)
data = urllib.request.urlopen(url2).read().decode("utf-8","ignore")
print(len(data))

# 用户代理池，希望对象服务器更难识别爬虫
# 使用多个浏览器标识
uapools=[
    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18363"
]

def UA():
    opener=urllib.request.build_opener()
    thisua = random.choice(uapools)
    ua = ("User-Agent",thisua)
    opener.addheaders=[ua]
    urllib.request.install_opener(opener)
    print("当前UA ："+str(thisua))

for i in range(0,10):
    UA()
    data = urllib.request.urlopen(url2).read().decode("utf-8","ignore")
    print(len(data))
```

简单实战：爬取糗事百科热门并翻页
