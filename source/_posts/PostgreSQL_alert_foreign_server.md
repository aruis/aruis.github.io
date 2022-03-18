---
title: PostgreSQL修改FDW相关配置
date: 2018-08-10 08:21:01
categories: 程序人生
tags:
    - PostgreSQL
---
必要的时候要修改`PostgreSQL`中配置的外部服务器。有个`ALTER SERVER`命令是专门应对这种场景的。比如我的外部数据源服务器地址换了，只需要改下之前配置的`host`地址即可，`SQL`如下
```
ALTER SERVER foreign_server OPTIONS (set host '192.168.0.88');
```
更多内容可以查看官方文档[http://www.postgres.cn/docs/9.6/sql-alterserver.html]