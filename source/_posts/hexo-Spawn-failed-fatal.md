---
title: hexo-Spawn-failed-fatal
tags: hexo
categories: server
abbrlink: 21614
date: 2020-02-13 00:00:31
---

今天偶尔发现博客部署到阿里云上没有成功
<!-- more -->
错误代码：
```bash
fatal: LF would be replaced by CRLF in 
2019/06/21/hello-world/index.html
FATAL Something's wrong. Maybe you can find the solution here: 
https://hexo.io/docs/troubleshooting.html
Error: Spawn failed
at ChildProcess.<anonymous> 
(D:\\Hexo\node_modules\hexo-util\lib\spawn.js:52:19)
at emitTwo (events.js:106:13)
at ChildProcess.emit (events.js:191:7)
at ChildProcess.cp.emit 
(D:\blog\Hexo\node_modules\cross-spawn\lib\enoent.js:40:29)
at Process.ChildProcess._handle.onexit 
(internal/child_process.js:215:12)
```

解决办法：
在.git的配置文件里添加

```bash
autocrlf = false
safecrlf = false
```
