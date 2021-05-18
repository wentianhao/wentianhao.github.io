---
title: Python下载文件的N种方法
katex: false
date: 2020-11-20 15:34:18
tags: python
categories: python
---

记录python下载文件的各种方法，从下载简单的小文件到用断点续传的方式下载大文件
<!-- more -->

## Requests

使用Requests模块的get方法从url上下载文件，在python爬虫中经常使用它来下载简单的网页内容

```python
import requests

# 图片url
url = 'https://cn.bing.com/th?id=OHR.DerwentIsle_EN-CN8738104578_400x240.jpg'

def requests_download():
    content = requests.get(url).content

    with open('pic_requests.jpg','wb') as file:
        file.wirte(content)
```

## urllib

使用urllib模块的urlretrieve方法直接将url请求保存成文件

```python
from urllib import request

# 图片url
url = 'https://cn.bing.com/th?id=OHR.DerwentIsle_EN-CN8738104578_400x240.jpg'

def urllib_download():
    request.urlretrieve(url,'pic_urllib.jpg')

```

## urllib3

urllib3是一个用于HTTP客户端的python模块，它使用连接池对网络进行请求访问

```python
def urllib3_download():
    # 创建一个连接池
    poolManager = urllib3.PoolManager()

    resp = poolManager.request('GET',url)

    with open('pic_urllib3.jpg','wb') as file:
        file.write(resp.data)
    
    resp.release_conn()
```

## wegt

在Linux系统中有wegt命令，下载网上资源，python中也有对应的wegt模块，使用`pip install wegt` 安装

```python
import wegt

# 图片url
url = 'https://cn.bing.com/th?id=OHR.DerwentIsle_EN-CN8738104578_400x240.jpg'

def wegt_download():
    wegt.download(url,out='pic_wegt.jpg')
```

OR `python -m wget https://cn.bing.com/th?id=OHR.DerwentIsle_EN-CN8738104578_400x240.jpg `

## 分块下载大文件

在文件比较大，电脑内存空间完全不够用的情况下，可用使用requests模块的流模式，默认情况下stream参数为False，文件过大会导致内存不足。stream参数为True的时候requests并不会立刻下载，只有在调用iter_content或者iter_lines遍历内容时下载

iter_content ：一块一块的遍历要下载的内容

iter_lines : 一行一行的遍历要下载的内容

```python
import requests

def stream_download():
# vscode 客户端
    url = 'https://vscode.cdn.azure.cn/stable/e5a624b788d92b8d34d1392e4c4d9789406efe8f/VSCodeUserSetup-x64-1.51.1.exe'
    with requests.get(url,stream=True) as r:
        with open('vscode.exe','wb') as file:
            # chunk_size 指定写入大小 每次写入1024*1024 bytes
            for chunk in r.iter_content(chunk_size=1024*1024):
                if chunk:
                    file.write(chunk)
```

## 进度条

在下载大文件时加上进度条美化下载界面，可实时知道下载网络速度和已经下载的文件大小，这里使用tqdm模块作为进度条显示，可使用`pip install tqdm`安装

```python
from tqdm import tqdm

def tqdm_download():
    url = 'https://vscode.cdn.azure.cn/stable/e5a624b788d92b8d34d1392e4c4d9789406efe8f/VSCodeUserSetup-x64-1.51.1.exe'

    resp = requests.get(url,stream=True)

    # 获取文件大小
    file_size = int(resp.headers['content-length'])

    with tdqm(total=file_size,unit='B',unit_scale=True,unit_divisor=1024,ascii=True,desc='vscode.exe') as bar:
        with requests.get(url,stream=True) as r:
            with open('vscode.exe','wb') as fp:
                for chunk in r.iter_content(chunk_size=512):
                    if chunk:
                        fp.write(chunk)
                        bar.update(len(chunk))
```

tqdm 参数说明：

- total : bytes，整个文件的大小
- unit='B' : 按 bytes 为单位计算
- unit_scale=True : 以 M 为单位显示速度
- unit_divisor=1024 : 文件大小和速度按 1024 除以，默认时按 1000 来除
- ascii=True : 进度条的显示符号，用于兼容 windows 系统
- desc='vscode.exe' : 进度条前面的文件名

示例
![tqdm](https://whh.plus/images/chunk.gif)

## 断点续传

HTTP/1.1 在协议的请求头中增加了一个名为Range的字段域，Range字段域让文件从已经下载的内容开始继续下载

如果网站支持Range字段域请求相应的状态码为206(Partial Content)，否则为416(Requested Range not satisfiable)

Range的格式

```bash
Range:[unit=first byte pos] - [last byte pos]，即 Range = 开始字节位置-结束字节位置，单位：bytes
```

将Range加入到headers中

```python
import os
from tqdm import tqdm

def resume_download():
    url = 'https://vscode.cdn.azure.cn/stable/e5a624b788d92b8d34d1392e4c4d9789406efe8f/VSCodeUserSetup-x64-1.51.1.exe'

    r = requests.get(url,stream=True)

    # 获取文件大小
    file_size = int(r.headers['content-length'])

    file_name = 'vscode.exe'
    # 如果文件存在获取文件大小，否则从0开始下载
    first_byte = 0
    if os.path.exists(file_name):
        first_byte = os.path.getsize(file_name)

    # 判断是否已经下载完成
    if first_byte >= file_size:
        return
    
    # Range 加入请求头
    header = {"Range" : f"bytes={first_byte}-{file_size}"}

    # 加入一个initial 参数
    with tqdm(total=file_size, unit='B', initial=first_byte, unit_scale=True, unit_divisor=1024, ascii=True, desc=file_name) as bar:
        # 加入 headers 参数
        with requests.get(url,headers=header,stream=True) as r:
            with open(file_name,'ab') as fp:
                for chunk in r.iter_content(chunk_size=512):
                    if chunk:
                        fp.write(chunk)
                        bar.update(len(chunk))
```

示例
![tqdm](https://whh.plus/images/tqdm.gif)