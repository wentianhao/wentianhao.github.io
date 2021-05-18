---
title: XPath表达式基础
date: 2020-02-19 23:07:22
tags:
    - python
    - python爬虫
categories: python
---
## 基础
除了正则表达式之外，还有一些非常好用的信息筛选工具，比如XPath表达式、BeautifulSoup等

- / 逐层提取
- text()提取标签下面的文本
- //标签名**
- //标签名[@属性='属性值']提取属性为XX的标签
- @属性名 代表取某个属性值
<!-- more -->
-----------
## 实例

```html
<html>
    <head>
        <title>主页</title>
    </head>
    <body>
        <p>abc</p>
        <p>bbbvb</p>
        <a href="//qd.alibaba.com/go/pcdetail" target="_top">安全推荐</a>
        <a href="//qd.alibaba.com/go/pcdetail" target="_top">安全推荐2</a>
        <div class="J AsyncDC" data-type="dr">
        <div id="official-remind">
            明月几时有
        </div> 
    </div>
</body>
```

XPath表达式 | 提取内容
:----: | :----:
/html/head/title/text() | 主页
//p/text() | ['abc','bbbvb']
//a | 提取所有的a标签
//div[@id='official-remind']/text() | 明月几时有
//a/@href | //qd.alibaba.com/go/pcdetail