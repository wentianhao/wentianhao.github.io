---
title: scrapy爬虫项目
date: 2020-02-29 13:25:55
tags:
    - python
    - python爬虫
categories: python
---
Scrapy模块是一个非常常用的爬虫框架模块，使用Scrapy可以快速创建爬虫项目。
<!-- more -->
**Scrapy模块的安装**

建议安装顺序
- 升级pip: python -m pip install --upgrade pip 

- 安装wheel pip install wheel

- 安装lxml (https://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud)选择对应的python版本以及win32 or 64

- 安装Twisted (https://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud)选择对应的python版本以及win32 or 64

+ 如果使用pycharm可以在Setting -> Project -> Project Interpreter 点击 + 添加最新的库

- pip install scrapy 或 pip install scrapy==1.1.0rc3

- 下载安装pywin32并配置 

![pywin32](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/pythoncom38.png?raw=true)

将这俩dll复制到system32(由于权限问题需手动确认一遍)


-----------
**Scrapy常用指令实战**

指令 | 意义
:---: | :---:
startproject | 创建爬虫项目
genspider -l | 查看爬虫模板
genspider -t | 模板 爬虫文件名 域名 创建爬虫
crawl | 运行爬虫
list | 查看有哪些爬虫

- 创建爬虫项目

![startproject](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/startproject.png?raw=true)

创建完成之后的项目目录:

![startproject](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/scrapy.png?raw=true)

scrapy.cfg: 项目配置文件

_init_.py ： 初始化文件

items.py : 定义目标信息

spiders ：存放多个爬虫文件

middlewares.py : 中间件

pipelines.py : 爬后处理(例如写入数据库...)

settings.py : 总体设置信息

- 指令测试

![指令测试](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/scrapy_g.png?raw=true)


------
**Scrapy爬虫项目编写基础**

- 创建爬虫项目

- 编写items

```python
title = scrapy.Field()
```

- 创建爬虫文件

- 编写爬虫文件

```python
# -*- coding: utf-8 -*-
import scrapy
from wh_first.items import WhFirstItem

class FstSpider(scrapy.Spider):
    name = 'fst'
    allowed_domains = ['aliwx.com.cn']
    start_urls = ['http://www.aliwx.com.cn/']

    def parse(self, response):
        item=WhFirstItem()
        item["title"] = response.xpath("//p[@class='title']/text()").extract()
        # print(item["title"])
        yield item
```

- 编写pipelines

```python
def process_item(self, item, spider):
    for i in range(0,len(item["title"])):
    print("------------")
    print(item["title"][i])
```

- 配置settings

```python
ITEM_PIPELINES = {
'wh_first.pipelines.WhFirstPipeline': 300,
}
```

**使用Scrapy编写当当网商品数据爬虫实战**

- 目标站点：[当当](http://category.dangdang.com/cp01.03.41.00.00.00.html)

- 需求数据：商品标题、链接、评论数等数据

- 要求：自动翻页并自动写入数据库

[源码](https://github.com/wentianhao/pyhton)