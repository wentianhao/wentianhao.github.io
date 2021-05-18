---
title: gem5与nvmain混合编译（二）———配置和运行ｐａｒｓｅｃ-2.1
date: 2020-01-13 10:47:24
tags: [gem5,nvmain,parsec-2.1] 
categories: gem5
---
## **参考资料：**
[**官方资料**](http://www.gem5.org/PARSEC_benchmarks)：http://www.gem5.org/PARSEC_benchmarks
http://pfzuo.github.io/2016/06/06/Configure-and-run-parsec-2.1-benchmark-in-GEM5/

配置ＰＡＲＳＥＣ Ｂenchmark (以ＡＲＰＨＡ架构为例)
<!-- more -->
1. 在gem5目录下新建一个文件夹用来存储ＰＡＲＳＥＣ Ｂenchmark 的disk image

```bash
mdkir fs_images
cd fs_images
```

2. 下载初始的系统文件

```bash
wget http://www.m5sim.org/dist/current/m5_system_2.0b3.tar.bz2
tar jxvf m5_system_2.0b3.tar.bz2
mv m5_system_2.0b3 system
```
解压后，文件目录结构：

```bash
system/
binaries/
      console
      ts_osfpal
      vmlinux
disks/
      linux-bigswap2.img
      linux-latest.img
```

3. 下载ＰＡＲＳＥＣ Benchmark相关文件，并替换掉system 文件夹中的对应文件

- 下载ＰＡＲＥＳＣ对应的linux kernel文件，替换 `system/binaries/vmlinux`

```bash
cd ./system/binaries/
wget http://www.cs.utexas.edu/~parsec_m5/vmlinux_2.6.27-gcc_4.3.4
rm vmlinux
mv vmlinux_2.6.27-gcc_4.3.4 vmlinux
```

- 下载对应的ＰＡＬ code文件，替换`system/binaries/ts_osfpal`

```bash
wget http://www.cs.utexas.edu/~parsec_m5/tsb_osfpal
rm ts_osfpal
mv tsb_osfpal ts_osfpal
```

- 下载ＰＡＲＳＥＣ-2.1 Ｄisk Image 并解压

```bash
cd ../disks/
wget http://www.cs.utexas.edu/~parsec_m5/linux-parsec-2-1-m5-with-test-inputs.img.bz2
bzip2 -b linux-parsec-2-1-m5-with-test-inputs.img.bz2
```

4. 进入gem5文件夹，修改两个文件（SysPaths.py 和 Benckmarks.py）配置parsec的路径和文件名

打开SysPaths.py配置parsec disk image的完整路径：

```bash
vim ./configs/common/SysPaths.py 
```

修改前：

```bash
path = [ ’/dist/m5/system’, ’/n/poolfs/z/dist/m5/system’ ]
```

修改后：

```bash
path = [ ’/dist/m5/system’, ’/path/to/gem5/fs_images/system’ ]
```

打开Benchmarks.py，修改image文件名：

```bash
vim ./configs/common/Benchmarks.py
```

修改前：

```bash
elif buildEnv['TARGET_ISA'] == 'alpha':
return env.get('LINUX_IMAGE', disk('linux-latest.img'))
```

修改后：

```bash
elif buildEnv['TARGET_ISA'] == 'alpha':
return env.get('LINUX_IMAGE', disk('linux-parsec-2-1-m5-with-test-inputs.img'))
```

5. 生成benchmark的script文件，用于运行benchmark

下载PARSEC script生成包，并解压到gem5目录下即可：

```bash
wget http://www.cs.utexas.edu/~parsec_m5/TR-09-32-parsec-2.1-alpha-files.tar.gz
tar zxvf TR-09-32-parsec-2.1-alpha-files.tar.gz
```

生成script命令：

```bash
./writescripts.pl <benchmark> <nthreads>
```

ＰＡＲＳＥＣ 有13 种Ｂenchmark:

```bash
blackscholes
bodytrack
canneal
dedup
facesim
ferret
fluidanimate
freqmine
streamcluster
swaptions
vips
x264
rtview
```

例如生成x264 script命令

```bash
cd TR-09-32-parsec-2.1-alpha-files/
./writescripts.pl x264 1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519103138529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)
 6. 根据生成的script文件运行gem5 

```bash
./build/ALPHA/gem5.opt ./configs/example/fs.py -n <number> --script=./path/to/runScript.rcS --caches --l2cache
```

7. 新开一个终端，使用telnet 与gem5模拟系统进行交互

```bash
telnet localhost 3456
```

如果使用这种方式连接发生意外中断，推荐使用以下交互方式
使用m5term

```bash
cd gem5/util/term/
make
sudo make install
sudo ./m5term 127.0.0.1 3456
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519104051256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19pbmRvd3M=,size_16,color_FFFFFF,t_70)