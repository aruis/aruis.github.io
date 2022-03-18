---
title: PortgreSQL中找到那些被大量顺序扫表没走索引的表
date: 2019-02-25 14:18:06
categories: 程序人生
tags:
    - PostgreSQL
---

```
select schemaname,
       relname,
       seq_scan,
       seq_tup_read,
       seq_tup_read / seq_scan as avg,
       idx_scan
from pg_stat_user_tables
where seq_scan > 0
order by seq_tup_read desc
limit 20
```
解释一下
* `seq_scan`是表上发生顺序扫描的次数
* `seq_tup_read`是顺序扫描时系统读取了多少个元组
* `idx_scan`是表上发生索引扫描的次数

通过上面的`SQL`就能查询到那些被频繁访问，但是几乎没有利用到索引的表，这样我们就可以针对这些表针对性的创建索引，从而大幅提升数据库的访问速度。

补充一点：顺序扫表不是一定不好，但是大规模的顺序扫表常常是数据库性能低下的根源。