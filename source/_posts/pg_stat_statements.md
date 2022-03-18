---
title: 利用pg_stat_statements排查PostgreSQL中的慢SQL
date: 2019-03-24 21:26:07
categories: 程序人生
tags:
    - PostgreSQL
---
1. 编辑`postgresql.conf`
    * `shared_preload_libraries = 'pg_stat_statements'`
    * `track_activity_query_size = 16384`
2. 启用扩展`create extension pg_stat_statements`
3. 查找慢`SQL`
```
select round((100 * total_time / sum(total_time) over ())::numeric, 2) percent,
       round(total_time::numeric, 2) as                                total,
       calls,
       round(mean_time::numeric, 2)  as                                mean,
       query
from pg_stat_statements
order by total_time desc;
```

