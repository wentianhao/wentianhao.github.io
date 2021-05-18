---
title: Anaconda
date: 2020-03-31 15:51:19
tags: anaconda
categories: python
---

1.查看创建好的环境：
```bash
conda info --envs
```

2.激活/进入环境
```bash
activate env_name
```
<!-- more -->
3.退出当前环境到base
```bash
conda deactivate
```

4.新建一个环境
```bash
conda create --n name python=3.6
```

5.删除一个环境
```bash
conda remove -n name --all
```

6.环境重命名

通过克隆旧环境，再删除旧环境得到
```bash
# 进入旧环境
activate old
# 克隆
conda create -n new --clone old
# 退出旧环境
conda deactivate
#删除旧环境
conda remove -n old --all
```