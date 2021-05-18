---
title: Notebook嵌入Hexo博客
katex: false
date: 2020-09-25 23:43:18
updated: 2020-09-25 23:43:18
tags:
    - jupyter notebook
    - hexo
categories: server
---

## 因

由于最近在notebook上学pytorch，之前就觉得notebook来写博客贼棒，然后就想着把pytorch的学习过程给完整的在hexo博客上展示出来，记录一下
<!-- more -->
## 使用github+nbviewer分享notebook

Jupyter的实例Jupyter的.ipynb文件显示加载Hexo个人博客静态页面中嵌入Jupyter

1.将ipynb文件发布到GitHub

过程不赘述，若不会百度之

2.将GitHub上的分享链接发布到[nbviewer](https://nbviewer.jupyter.org/)

ex: [pytorch学习笔记](https://github.com/wentianhao/wentianhao.github.io/blob/master/notebooks/pytorch.ipynb) 

![nbviewer](https://whh.plus/images/nbviewer.jpg)

GO! 即可在线显示

3.Hexo个人博客静态页面中嵌入Jupyter

由于Markdown能够支持html语法，所以，稍微调整一下高度和宽度，就能够适应博客的页面。

```html
<iframe src="https://nbviewer.jupyter.org/github/wentianhao/wentianhao.github.io/blob/master/notebooks/pytorch.ipynb" width="570" height="2000"></iframe> 
```

效果如下:

<iframe src="https://nbviewer.jupyter.org/github/wentianhao/wentianhao.github.io/blob/master/notebooks/pytorch.ipynb" width="570" height="2000"></iframe> 


参考文章:

1.[Jupyter notebook简介及嵌入Hexo博客中](https://cloud.tencent.com/developer/article/1471240)

2.[使用github+nbviewer分享notebook结果](https://www.jianshu.com/p/ff48ea53b78a)