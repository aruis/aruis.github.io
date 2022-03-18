---
title: PostgreSQL打开SQL日志
date: 2018-08-22 07:25:04
categories: 程序人生
tags:
    - PostgreSQL
---
找到`PostgreSQL`安装目录所在的地方，编辑配置文件`postgresql.conf`
确保
```
log_destination = 'stderr'
```
这样可以打开日志功能
然后找到`atement`设置成`all`
```
log_statement = 'all'
```
重启服务之后就可以跟踪到所有的`SQL`语句了。

当然其他还有很多配置，可以进行更细致的定制，可以参考官方文档[错误报告和日志](http://www.postgres.cn/docs/10/runtime-config-logging.html)