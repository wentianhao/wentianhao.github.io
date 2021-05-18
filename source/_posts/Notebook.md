---
title: jupyter Notebook
date: 2020-01-19 15:46:26
tags: jupyter notebook
categories: server
---

## 什么是Jupyter Notebook?
Jupyter Notebook是一款开放源代码的web应用程序，可以让我们创建并共享代码和文档。

它提供了一个环境，你可以在其中记录代码，运行代码，查看结果，可视化数据并在查看输出结果。这些特性使其成为一款执行端到端数据科学工作流程的便捷工具 ，可以用于数据清理，统计建模，构建和训练机器学习模型，可视化数据以及许多其他用途。

<!-- more -->

## ubuntu安装Jupyter Notebook

- 安装Anaconda

```bash
mkdir anaconda 
#创建文件夹
cd anaconda #进入文件夹
wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh #从指定地址下载文件
bash Anaconda3-4.4.0-Linux-x86_64.sh #安装文件
```

检查是否安装成功
```bash
conda --version
```

如果出现conda: command not found

添加环境变量：

```bash
vim ~/.bashrc
```

在最后加上：anaconda3的路径

```bash
export PATH=$PATH:/your/path/to/anaconda3/bin
```

保存更改，运行

```bash
source ~/.bashrc
```

检查是否安装成功

```bash
conda --version
```

conda创建python虚拟环境

```bash
conda create -n your_env_name
source activate your_env_name #进入虚拟环境
python --version #更换到python3
echo alias python=python3 >> ~/.bashrc
source ~/.bashrc
python --version
```

如果anaconda版本不够，可手动更新升级 (具体可百度)

- 通过anaconda安装jupyter及各种包

```bash
conda install jupyter
conda update --all #升级所有包
```

-   配置远程访问登录密码

1、在终端输入

```bash
[root@localhost]:~#ipython
In [1]: from notebook.auth import passwd 
In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:xxxxx'
In [3]: exit
```

解释：这是在python中调用passwd()方法，输入密码输出hashcode编码

2、保存刚刚的hashcode

```bash
jupyter notebook --generate-config --allow-root #生成配置文件
vim ~/.jupyter/jupyter_notebook_config.py #打开配置文件```
```

如果没使用过vim的，可以学一下。

在配置文件中,加入:

```bash
c.NotebookApp.ip='*'     # * 代表所有iP都能访问 ，也可以指定ip
c.NotebookApp.password = u'sha1:xxxxx'  # 刚才复制的那个密文
c.NotebookApp.open_browser = False  # 禁止自动打开浏览器
c.NotebookApp.port =8888    #指定一个端口

c.NotebookApp.notebook_dir = 'PATH'  #指定工作空间
c.PAMAuthenticator.encoding = 'utf8'         #指定utf-8编码，解决读取中文路径或者文件乱码问题
```

如果需要取消jupyter notebook登录密码

```bash 
# 将c.NotebookApp.password=u'sha1:xxx'注释掉
# c.NotebookApp.password=u'sha1:xxx'
c.NotebookApp.token=''
```
:wq保存

【注意】：到目前为止jupyter就已经在服务器中安装配置好了，但是8888端口默认是关闭的，如果是阿里云服务器，需要在阿里云控制台开放

8888端口

- 最后输入 

```bash
ufw disable
```

关闭防火墙

```bash
jupyter notebook --ip=0.0.0.0 --allow-root #在服务器运行jupyter notebook
```

- 浏览器输入http://xxx:8888 即可登录

- 后台运行

```bash
nohup jupyter notebook --ip=0.0.0.0 --allow-root > /home/whh/notebook/jupyter.log 2>&1 &
```

&让命令后台运行，并输出到log中

nohup 表示 no hang up ，不挂起

- 终止进程

```bash
ps -aux | grep jupyter #查看运行的jupyter进程
kill -9 pid
```