---
title: 用pg_cron定时刷新PostgreSQL的物化视图
date: 2018-08-11 07:04:30
categories: 程序人生
tags:
    - PostgreSQL
---
在`PostgreSQL`中可以很轻松的创建物化视图，但是却没有自动刷新物化视图的机制。通常来说，不外乎两种方式，一种是通过触发器，另一种就是定时任务调度。今天我们就来说说第二种方式。
主要借助一个名为[pg_cron](https://github.com/citusdata/pg_cron)的扩展。
安装方法在官方介绍里面已经说的很清楚了，不再赘述，这里提醒一点，安装完后，是需要修改`postgresql.conf`配置文件，并重启`PostgreSQL`服务的。具体修改如下：
```
shared_preload_libraries = 'pg_cron'
cron.database_name = 'postgres'
```
第二行的指定`cron`的元数据相关信息存放的数据库，是可以改成其他的。这里要明确一个概念，`cron`安装的数据库，和它要控制的数据库没有什么必然联系，并不因为说安装在了`postgres`库，就不能调度其他库了，这个在后续具体配置的时候，就能明白了。
做完上述步骤，保证`PostgreSQL`服务重启过后，就可以用`psql`或其他工具连接到`cron.database_name`对应的数据，执行
```
CREATE EXTENSION pg_cron;
```
如果不在指定的库执行的话，会遇到错误
```
Jobs must be scheduled from the database configured in cron.database_name, since the pg_cron background worker reads job descriptions from this database.
```
届时注意即可。
一切准备就绪后，就可以使用了，使用方式相当简单。这里我用一个查询`now()`的物化视图做演示
```
create materialized view now as select now();
SELECT cron.schedule('26 * * * *', 'refresh materialized view now;');
```
这就实现了每小时26分的时候去刷新物化视图，也就是意味着，任意时候执行
```
select * from now;
```
获得的结果，都是距离此刻最近的26分，而不是当前时间。
那么，已经创建的任务，该如何管理呢。其实很简单，定时任务数据都存放在`cron.job`表中，我们看看里面的数据就明白了。
```
+---------+------------+--------------------------------+------------+------------+------------+------------+
| jobid   | schedule   | command                        | nodename   | nodeport   | database   | username   |
|---------+------------+--------------------------------+------------+------------+------------+------------|
| 1       | 26 * * * * | refresh materialized view now; | localhost  | 5432       | analyze    | postgres   |
+---------+------------+--------------------------------+------------+------------+------------+------------+
```
如果要取消一个任务，就需要拿到`jobid`，然后执行下面这句就好了。
```
select cron.unschedule(1)
```
