---
title: hexo-to-your-server
tags:
  - hexo
  - server
  - nginx
categories: server
abbrlink: 46941
date: 2020-01-20 15:20:32
---

## 服务器配置
---
**搭建远程git私库**

- 登录远程服务器，安装git

```bash
git --version #查看是否安装
```

-  创建用户并配置仓库
<!-- more -->
```bash
useradd git #创建用户
passwd git #设置密码
mkdir /home/git #创建git文件夹
chown -R git:git /home/git
su git #授权
cd /home/git/
mkdir -p projects/blog #项目存在的真实目录
mkdir repos && cd repos
git init --bare blog.git #创建一个裸露的仓库
cd blog.git/hooks
vim post-receive #创建hook钩子函数,输入
```

```bash
#!/bin/sh
git --work-tree=/home/git/projects/blog --git-dir=/home/git/repos/blog.git checkout -f
```

添加完成后修改权限

```bash
chmod +x post-receive
exit //退出到root登录
chown -R git:git /home/git/repos/blog.git #添加权限
```

- 测试git仓库是否可用

在本地电脑的git中输入

```bash
git clone git@server_ip:/home/git/repos/blog.git
```

如果能把空仓库拉下来，则说明git仓库搭建成功

![pic](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzE0LzE2ZDJiNmVhNGRmNTQxNzg?x-oss-process=image/format,png)

- 建立ssh信任关系，在本地电脑

如果没有rsa密钥则生成

```bash
git config --global user.name "yourname"
git config --global user.email youremail@example.com
ssh-keygen -t rsa -C "youremail@example.com"
git config --global core.autocrlf false  # 禁用自动转换，这个不设置后面上传时会出现警告，如下
```

![pic](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzE0LzE2ZDJiNmU1MWU4MGFjMTU?x-oss-process=image/format,png)

建立ssh信任关系

```bash
ssh-copy-id -i C:/Users/yourname/.ssh/id_rsa.pub git@server_ip
ssh git@server_ip #测试能否登录
```

注意：此时ssh登录git用户不需要密码!如果需要则没配置好

- 禁用git 用户的shell登录权限，从而只能用git clone,git push等功能

```bash
cat /etc/shells #查看git-shell 是否在登录方式里
which git-shell #查看是否安装
vi /etc/shells
```

添加第二步显示的路径，通常为 /usr/bin/git-shell

- 修改/etc/passwd中的权限

```bash
#将原来的 
git:x:1000:1000::/home/git:/bin/bash

#修改为
git:x:100:1000:,,,:/home/git:/usr/bin/git-shell
```


---
**nginx安装**

```bash
apt-get install nginx
```

**验证安装**
```python
nginx # 启动nginx 
```

查看是否安装成功，在浏览器输入公网ip

**修改配置文件**

```bash
# 查找nginx安装路径
whereis nginx 
cd /etc/nginx
vim nginx.conf
```

![code](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/nginx_blog.png?raw=true)

![code](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/nginx_blog2.png?raw=true)

----
## 发布

**配置_config.yml文件**

1.编辑_config.yml的deploy属性

![code](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/deploy.png?raw=true)

2.在package.json中添加npm脚本

```json
"scripts":{
"deploy":"hexo clean && hexo g -d",
"start": "hexo clean && hexo g && hexo s"
},
```

3.使用[ProtableHexo](https://whh.plus/2020/01/12/hello-world/)自带的脚本上传