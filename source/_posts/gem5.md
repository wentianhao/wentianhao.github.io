---
title: gem5的安装与使用
tags: gem5
categories: gem5
abbrlink: 9963
date: 2020-01-13 10:41:13
---


## gem5的安装与使用

**1. 安装各类库文件（ubuntu）**

```bash
sudo apt-get install mercurial scons swig gcc m4 python python-dev 
libgoogle-perftools-dev g++ libprotobuf-dev
```
由于ubuntu缺省情况下，并没有提供C/C++的编译环境，单独安装gcc以及g++比较麻烦，ubuntu提供一个build-essential软件包。安装这个软件包，编译C/C++需要的软件包都会被安装

```bash
sudo apt-get install build-essential
```
<!-- more -->
**2.下载gem5源码**

```bash
hg clone http://repo.gem5.org/gem5
```


![下载gem5](https://img-blog.csdnimg.cn/20190420183524941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190420183640443.png)

- 配置hgrc
打开hgrc文件：`vim gem5/.hg/hgrc`,将以下文件内容拷贝到里面，修改username,style等信息，style是你gem5文件里的style.其实我觉得`[email] `中的内容其实作用不大，但是看了好多其他的都差不多是这样我自己就只修改了style和username。
信息的修改只是一种标识，不影响编译和使用，最重要的是`[extensions] hgext.mq =`   扩充hg命令.

```bash
[ui]
# Set the username you will commit code with
username=Your Name <your@email.address>
ssh = ssh -C
# Always use git diffs since they contain permission changes and rename info
[defaults]
qrefresh = --git
email = --git
diff = --git
[extensions]
# These are various extensions we find useful
# Mercurial Queues -- allows managing of changes as a series of patches
hgext.mq =
# PatchBomb -- send a series of changesets as e-mailed patches
hgext.patchbomb =
# External Diff tool (e.g. kdiff3, meld, vimdiff, etc)
hgext.extdiff =
# Fetch allows for a pull/update operation to be done with one command and automatically commits a merge changeset
hgext.fetch =
# Path to the style file for the M5 repository
# This file enforces our coding style requirements
style = /path/to/your/gem5/util/style.py
[email]
method = smtp
from = Your Name <your@email.address>
[smtp]
host = your.smtp.server.here
```

**3.获取NVMain**

- 注册bitbucket帐号
https://bitbucket.org/account/signup/
- 获取NVMain使用权
打开https://bitbucket.org/mrp5060/nvmain/并登陆bitbucket账号，登陆后重新打开前面这个网站获取下载命令 

```bash
git clone https://your account@bitbucket.org/xxx
```

- [或者点击下载nvmain（无需注册）](https://bitbucket.org/mrp5060/nvmain/downloads/)
- nvmain文件位置随意，我放在了gem5根目录下，如果放的位置不同，后面编译时注意相对位置

4.**安装NVMain补丁**

- 进入gem5根目录

- 初始化补丁目录

```bash
hg init
hg qinit
```

- 输入补丁 (注意nvmain文件位置，以及补丁的版本号)

```bash
hg qimport -f ./nvmain/patches/gem5/nvmain2-gem5-11688+
```

  1. 安装补丁

```bash
hg qpush
```

 2. 如果打补丁出错,检查gem5的版本，检查nvmain中补丁的版本

```bash
# 版本不对，更新gem5
hg update 11688
# 检查补丁是否打上
hg qapplied
```


5.**联合编译gem5和nvmain**

```bash
scons EXTRAS=nvmain ./build/X86/gem5.opt 
```

![scons done](https://img-blog.csdnimg.cn/20190420184756987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

**在这一步中容易出现各种错误**

**1. ImportError: No module named gem5_scons:**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190420190508767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

查看gem5目录中是否存在gem5_scons文件夹，如果没有则在site_scons中将gem5_scons移到gem5根目录下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190420190758359.png)

**2. [-Werror=implicit-fallthrough=]**

这个问题困扰了我好久，百度了好久终于发现问题

如果gcc版本号为7，那么源代码的switch-case块中如果忘了加上break，有可能会报错Implicit fallthrough error。

**解决办法：编译时忽略Implicit fallthrough error这个错误**

**修改 gem5中的SConstruct**

在**CCFLAGS**中添加 **-Wno-implicit-fallthrough** 即可忽略该错误

```bash
main.Append(CCFLAGS=['-Wall', '-Wundef', '-Wextra',
                        '-Wno-sign-compare', '-Wno-unused-parameter','-Wno-implicit-fallthrough'])
```

**3. [-Werror]相关类似错误**

均可按照上面的方法，忽略gcc-7版本的问题导致的错误

6.**测试**

```bash
./build/X86/gem5.opt ./configs/example/se.py -c ./tests/test-progs/hello/bin/x86/linux/hello
```

![hello world](https://img-blog.csdnimg.cn/20190420194540619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)

如果遇到和我一样的错误的，希望对你有帮助，么么哒