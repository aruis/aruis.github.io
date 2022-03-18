---
title: 找出Postgres中占用磁盘最多的表
date: 2020-10-30 14:15:10
categories: 程序人生
tags:
    - PostgreSQL
---

```sql
SELECT schemaname,
       relname,
       pg_size_pretty(pg_table_size(relid))
from pg_stat_user_tables
order by pg_table_size(relid) desc ,schemaname;
```