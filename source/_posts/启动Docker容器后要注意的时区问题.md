---
title: 启动Docker容器后要注意的时区问题
date: 2018-07-30 08:46:14
categories: 程序人生
tags: Docker
---
对于中国用户来说，一般的docker容器启动后，如果执行`docker  exec -it xxxx date`会发现打印出来的时间，比当前北京时间早八个小时。所以需要调整容器的时区，主要有两个命令（要在容器内部执行）：
* `cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
* `echo "Asia/Shanghai" > /etc/timezone`

其中第一个命令就能解决不少问题，比如我用`PostgreSQL`的话，`select now()`就可以通过第一条命令修正。但是`Tomcat`等java相关的程序，还需要第二条命令，才能获得正确的时间。