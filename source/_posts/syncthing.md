---
title: 数据同步神器
date: 2020-03-02 16:58:59
tags: sync
categories: server
---

![syncthing](https://www.moewah.com/usr/uploads/2019/03/3082024495.jpg)

最近打算同步一下数据，发现了一个口碑还不错的数据同步工具——Syncthing,是一款开源免费的数据同步工具，支持Android、Linux、Windows、Mac OS等系统，可以在任意2台设备之间，实现实时同步。通信使用TLS进行保护，所使用的加密包括完美的前向保密，很适合搭建私有同步网盘
<!-- more -->
## **安装**

本文只做Linux服务器和Windows电脑之间的同步安装以及演示操作。

### **1、linux安装Syncthing**
安装Syncthing:

```bash
#目前最新版v1.2.0，日后自行修改版本号
wget https://github.com/syncthing/syncthing/releases/download/v1.3.4/syncthing-linux-amd64-v1.3.4.tar.gz
tar xzvf syncthing-linux-amd64-v1.3.4.tar.gz
cp syncthing-linux-amd64-v1.3.4/syncthing /usr/local/bin
```

#### **开放端口**

Syncthing 默认监听以下几个端口

```bash
22000(TCP)
8384(TCP)
21027(UDP)
44647(UDP)
37269(UDP)
```

个人使用，仅开放22000(TCP)和8384(TCP)

8384:web访问控制端口
22000：节点访问端口

开启Syncthing

```bash
syncthing
```

第一次启动会生成配置，内网地址是进不去的，需修改成外网地址。

修改配置

```bash 
vim /root/.config/syncthing/config.xml
# 将127.0.0.1:8384改为
# 0.0.0.0:8384

#再次启动Syncthing
syncthing
```

浏览器访问:

http://server_ip:8384

需自行开放8384端口哦

设置账号密码，其他参数暂时无需更改，重启

![syncthing](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/HU.png?raw=true)

linux ：后台运行Syncthing

```bash
nohup synchthing & /dev/null &
```

### **2、Windows安装Syncthing**

[官网下载](https://syncthing.net/downloads/)对应版本的Syncthing,运行syncthing.exe

![cmd](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/syncmd.png?raw=true)

![setting](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/syncsetting.png?raw=true)

### **3、使用**

在linux的页面上添加Windows的远程设备ID，自行选择指定文件夹。可在添加设备->高级->地址列表，将dynamic 填写为 tcp://server_ip:22000

设备添加完成，即可同步文件夹。
