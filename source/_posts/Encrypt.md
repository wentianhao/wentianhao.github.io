---
title: Let's Encrypt 申请免费的Https证书
date: 2020-02-22 17:38:45
tags: https
categories: server
---
## Let's Encrypt证书
cerbot可以通过简单的命令来生成证书是免费的，而且还支持通配符证书，通配符证书指的是一个可以被多个子域名使用的公钥证书，多个子域名使用起来十分方便。申请和配置的流程简单，有效期90天，可以通过脚本更新证书。 
<!-- more -->
-------
## **Cerbot**
cerbot可以通过简单的命令来生成证书

```bash
git clone https://github.com/certbot/certbot
```

---------
## **申请证书**
在申请Let's Encrypt证书的时候，需要校验域名的所有权，证明操作者有权利为该域名申请证书，目前支持三种验证方式：
- dns-01:给域名添加一个DNS TXT 记录
- http-01:在域名对应的web服务器下放置一个HTTP well-known URL资源文件
- tls-sni-01:在域名对应的web服务器下放置一个HTTPS well-known URL资源文件

对于通配符域名只能通过dns-01的方式申请，在阿里购买的域名需要在阿里云的解析设置中添加解析记录。使用下面的命令生成证书，注意将*.example.com 和 example.com替换成自己的域名

```bash
cd cerbot
./cerbot-auto certonly --manual \ 
-d *.whh.plus \ 
-d whh.plus --agree-tos \ 
--manual-public-ip-logging-ok --preferred-challenges \ 
dns-01 --server https://acme-v02.api.letsencrypt.org/directory 
```

输入完上面命令之后，需要下载一堆依赖库，然后输入邮箱：

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): xxxxxxx@email.com
```

验证域名所有权：

```bash
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

    mhumL1xJOHPIZtFTEm4rotjJnR9TdkBVPuCS9YHvNjs

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

打开域名添加解析记录，记录类型选择TXT，主机记录输入_acme-challenge.example.com,记录值输入随机生成的字符串mhumL1xJOHPIZtFTEm4rotjJnR9TdkBVPuCS9YHvNjs。

-----
## **配置Https访问**
nginx.conf

```bash
# 设置 http 自动跳转到 https
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;                                      
}

# 监听 443 端口，转发请求到 3000 端口
server {
    listen 443;
    server_name example.com;
    location / {
        #选择的服务端口
        #proxy_pass http://127.0.0.1:3000;
        # blog地址，参照部署那篇文章
    }

    # 开启 ssl 并指定证书文件和秘钥的位置
    ssl on;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;        
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;          
}                                                                                     
```

## 续期

1.下载letsencrypt
```bash
git clone https://github.com/letsencrypt/letsencrypt
```

2.关闭nginx服务器
```bash
service nginx stop
```

3.执行更新证书脚本
```bash
./letsencrypt-auto certonly --standalone --email XXXX@qq.com -d XXX.XXX.com
```

--email 是设置的邮箱

-d 是设置对应的域名目录 

当消息出现 “IMPORTANT NOTES:

- Congratulations! ”

说明续期成功！

4.更改文件夹名

将新生成的文件夹改为nginx配置文件中的文件夹名
```bash
mv whh.plus-00001 whh.plus
```

5.重新开启nginx服务器
```bash
service nginx start
```

------------------
nginx配置: [https://whh.plus/2020/01/20/hexo-to-your-server/](https://whh.plus/2020/01/20/hexo-to-your-server/)


-----------------------------------------------------------
## 替代方法

最近在重新配置letsencrypt的时候，出现了如下错误:

```bash
Skipping bootstrap because certbot-auto is deprecated on this system.
Your system is not supported by certbot-auto anymore.
Certbot cannot be installed.
Please visit https://certbot.eff.org/ to check for other alternatives.
```
在github中certbot的(release)[https://github.com/certbot/certbot/releases]更新中说明

certbot-auto不再支持所有的操作系统！根据作者的说法，certbot团队认为维护certbot-auto在几乎所有流行的UNIX系统以及各种环境上的正常运行是一项繁重的工作，加之certbot-auto是基于python 2编写的，而python 2即将寿终正寝，将certbot-auto迁移至python 3需要大量工作，这非常困难，因此团队决定放弃certbot-auto的维护

替代方法：使用基于snap的新的分发方法

1.安装snap

```bash
apt-get install snapd
```

2.添加路径
```bash
echo "export PATH=$PATH:/snap/bin" >> ~/.bashrc #bash
source ~/.bashrc
echo "export PATH=$PATH:/snap/bin" >> ~/.zshrc #zsh
source ~/.zshrc
```

3.将snap更新至最新版本
```bash
snap install core
snap refresh core
```

4.安装certbot
```bash
snap install --classic certbot
```

5.生成证书
确保nginx处于运行状态，需要获取证书的站点在80端口，并且可以正常访问。

```bash
certbot certonly --nginx --email xxx@mail.com -d a.do.com
```

更新nginx配置并重启nginx
```bash
service nginx stop
nginx -c etc/nginx/nginx.conf
nginx -s reload
service nginx start
```

nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)问题的解决
```bash
fuser -k 80/tcp # 将进程杀死后，启动nginx
``` 
or

```bash
ps -A | grep nginx    # 杀死对应的两个进程之后，启动nginx
kill -9 pid1
kill -9 pid2 
```


6.更新证书
```bash
certbot renew
```


参考以及感谢: 

- [https://juejin.im/post/5c2aefb3f265da6157059430](https://juejin.im/post/5c2aefb3f265da6157059430)
- [https://jingyan.baidu.com/article/ff42efa9fc231fc19e220218.html](https://jingyan.baidu.com/article/ff42efa9fc231fc19e220218.html)
- [https://blog.csdn.net/zhangpeterx/article/details/104093318](https://blog.csdn.net/zhangpeterx/article/details/104093318)
- [https://blog.csdn.net/Dancen/article/details/112571444](https://blog.csdn.net/Dancen/article/details/112571444)
- [https://blog.csdn.net/gwdgwd123/article/details/104068563](https://blog.csdn.net/gwdgwd123/article/details/104068563)
- [https://blog.csdn.net/qq_27252133/article/details/53646986](https://blog.csdn.net/qq_27252133/article/details/53646986)