---
title: Request模块爬虫
tags:
  - python
  - python爬虫
categories: python
abbrlink: 36610
date: 2020-02-28 18:57:05
---

**Request模块的基础使用**

- 安装request

```python 
pip install requests
```

- 使用方法
    + 请求方式： get、post、put...
    + 参数：
    
        参数 | 含义
        :---: | :----:
        params |  get请求的参数
        headers | 伪装浏览器添加头信息
        proxies | 添加代理
        cookies | 添加cookie
        data | post请求
    + 属性：
        
        属性 | 含义
        :---: | :---:
        text | 对应请求响应数据
        content | 二进制类型的相应数据
        encoding | 网页编码
        cookies | 相应cookie
        url | 当前请求的url
        status | 状态码
<!-- more -->

```python
import requests
import re

# 伪装浏览器
hd = {"user-agent":"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36"}

# 添加代理
px = {"http":"http://127.0.0.1:8888"}

# 添加cookies
cookies= {"":""}

# get请求
rst = requests.get("https://www.aliwx.com.cn/",headers=hd,proxies=px)
print(rst)
title = re.compile("<title>(.*?)</title>",re.S).findall(rst.text)
print(title)
# www.baidu.com/s?wd="阿里文学"
pr = {"wd":"阿里文学"}
rst2 = requests.get("http://www.baidu.com/s",params=pr)
print(rst2)

# text 、 content、 encoding、url、status_code
print(rst2.text)
print(rst2.content)
print(rst2.encoding)
print(rst2.url)
print(rst2.status_code)

# cookie 转化为字典
cookie  = requests.utils.dict_from_cookiejar(rst2.cookies)
print(cookie)

# post请求
postdata = {"name":"www","pass":"123"}
rst3 = requests.post("https://www.iqianyue.com/mypost/",data=postdata)
print(rst3.text)
```

------
**使用requests模块实现云栖社区博文爬虫实战**

目标站点：[云栖社区](https://yq.aliyun.com/)

需求数据：指定关键词的博文数据

要求：自动翻页并下载到本地

分析：

第二页url：https://yq.aliyun.com/search/articles/?spm=a2c4e.11163080.searchblog.155.44fb2ec1RCP5cT&q=python&idx=default&days=&p=2

第三页url：https://yq.aliyun.com/search/articles/?spm=a2c4e.11163080.searchblog.165.4a4f2ec1GkHbdt&q=python&idx=default&days=&p=3

spm、idx、days测试去掉有无影响 (无影响)

第二页有效url：https://yq.aliyun.com/search/articles/?q=python&p=2

第三页有效url: https://yq.aliyun.com/search/articles/?q=python&p=3

源码：

```python
# 目标站点：[云栖社区](https://yq.aliyun.com/)
#
# 需求数据：指定关键词的博文数据
#
# 要求：自动翻页并下载到本地

import requests
import re
import time

key = "python"
url = "https://yq.aliyun.com/search/articles/"
data = requests.get(url,params={"q":key}).text
pat1 = '<div class="_search-info">找到(.*?)条关于'
allline=re.compile(pat1,re.S).findall(data)[0]
allpage=int(allline)//15+1
for i in range(0,1):
    print("-------正在爬第"+str(i+1)+"页-------------")
    index=str(i+1)
    getdata={"q":key,
            "p":index,
            }
    data=requests.get(url,params=getdata).text
    pat_url= '<div class="media-body text-overflow">.*?<a href="(.*?)">'
    articles=re.compile(pat_url,re.S).findall(data)
    for j in articles:
        thisurl="https://yq.aliyun.com"+j
        thisdata=requests.get(thisurl).text
        pat_title='<p class="hiddenTitle">(.*?)</p>'
        title=re.compile(pat_title,re.S).findall(thisdata)[0]
        pat_content='<div class="content-detail unsafe markdown-body">(.*?) <div class="copyright-outer-line">'
        content=re.compile(pat_content,re.S).findall(thisdata)[0]
        fh=open("./blog/"+str(title)+str(i)+"_"+str(time.time())+".html","w",encoding="utf-8")
        fh.write(title+"<br /><br />"+content)
        fh.close()
```