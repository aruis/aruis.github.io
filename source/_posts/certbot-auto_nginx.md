---
title: 使用certbot-auto给nginx加上免费https证书
date: 2018-09-19 13:42:43
categories: 程序人生
tags:
    - nginx
    - https
---
### 准备python3环境
```
# 非唯一方法，仅供参考
yum install rh-python36
scl enable rh-python36 bash
```

### 准备`certbot-auto`工具
```
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
```

### 准备nginx站点
```
yum install nginx
```
然后去`/etc/nginx/conf.d/`准备自己的站点配置文件，命名为`www.yourdomain.com.conf`
最简单的配置内容如下：
```
server {
        listen       80 ;
        server_name  www.yourdomain.com;
        root         /usr/share/nginx/html;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

### 运行`certbot-auto`
```
./certbot-auto --nginx
```
之后根据向导一步步操作即可，大概就是输入一下自己的有效邮箱，输入几个`Y`（同意），选一下刚创建的那个`nginx`配置文件就ok了。
如果出现
```
No supported Python package available to install. Aborting bootstrap!
```
可以尝试这个命令
```
./certbot-auto --nginx --no-bootstrop
```

### 定时更新证书
默认证书的有效期是90天，所以我们需要设置定时任务，隔断时间就`renew`下，通过`crontab -e`配置即可，内容如下
```
0 3 * * * /root/letsencrypt/certbot-auto renew --quiet
```
`renew`命令会在确定证书快要到期的时候，才真正`renew`证书的。

### 其他技巧：
* `debug`，访问https://letsdebug.net
* 查看证书情况（有效期），访问https://crt.sh