---
title: CentOS系统使用Shadowsocks搭建代理服务
date: 2018-07-19 08:48:00
categories: 实用技巧
tags: 
 - 代理
---
1. 确认pip是否安装，命令`pip help`，返回如图信息，说明已安装
![](/media/15319616591142.jpg)
如果返回下图，说明未安装
![](/media/15319617008516.jpg)
未安装需要执行以下子步骤：
    * `yum -y install epel-release`
    * `yum update`
    * `yum install python-pip`
2. 通过pip安装shadowsocks，命令`pip install shadowsocks`
3. 准备shadowsocks的配置文件，找个地方放就行，比如`/root/shadowsocks.json`，文件内容如下：
```
{
  "server": "0.0.0.0",
  "server_port": 1988,
  "local_address": "0.0.0.0",
  "local_port": 1080,
  "password": "xyz",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false,
  "workers": 5
}
```
    其中`server_port`、`passoword`、`method`三个参数比较重要，分别对应:代理服务所在端口、链接密码、加密方式，回头要用到。
4. 准备好配置文件之后，就可以启动shadowsocks服务了，命令为`ssserver -c /root/shadowsocks.json -d start`，至此shadowsocks服务启动完毕，然后就可以用客户端连接了。
5. 客户端下载地址在都在github上，这里给出最常用的mac版和windows版
    * [mac](https://github.com/shadowsocks/ShadowsocksX-NG/releases)
    * [windows](https://github.com/shadowsocks/shadowsocks-windows/releases)
6. 客户端安装完毕，就可以配置服务器连接了，这里给个参考配置：
    ![](/media/15319627604059.jpg)
地址就是服务器所在的地址，备注随意，剩下的三个配置，正好对应上面提过的配置。
7. 确定shadowsocks client是启动状态，就可以享受不一样的网络了。