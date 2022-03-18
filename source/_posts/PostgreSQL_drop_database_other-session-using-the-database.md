---
title: PostgreSQL删除数据库时提示other session using the database
date: 2018-08-07 08:06:26
categories: 程序人生
tags:
    - PostgreSQL
---
PostgreSQL中，如果想drop一个正在被人连接的数据库，是不可以的。提示如下：
```
postgres=# drop database ka;
ERROR:  database "ka" is being accessed by other users
DETAIL:  There is 1 other session using the database
```
应对方法是要通过`pg_terminate_backend`系统内置函数，把对应库在线连接给清理掉，使用方法如下：
```
SELECT pg_terminate_backend(pg_stat_activity.pid)
    FROM pg_stat_activity
    WHERE pg_stat_activity.datname = 'ka'
      AND pid <> pg_backend_pid();
```