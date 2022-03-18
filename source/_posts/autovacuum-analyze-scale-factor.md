---
title: PostgreSQL频繁更新数据的大表查询计划可能会搞错的应对方案
date: 2020-11-05 08:57:49
categories: 程序人生
tags:
    - PostgreSQL
---
当表的数据比较多（大概百万级以上）的时候，对表的使用是重度依赖analyze采集的数据的。尤其是当表处于被频繁的update、insert操作下，之前analyze的数据如果不及时更新，极有可能让查询计划走歪，然后导致一个查询可能要付出10倍以上的时间——我遇到过好几次，每次都是通过手动vaccum解决。
本来pg是有autovacuum的，这里之所以没有触发，还是因为表比较大，百万行的数据，update个几万行，变更率才百分之几，而调度autovacuum_analyze的默认阈值是百分之十。
这就比较尴尬了，好在pg支持针对单表做详细的定制。这里给出一个参考SQL：
```sql
alter table demo_table SET (fillfactor=85, autovacuum_vacuum_scale_factor=0.02, autovacuum_analyze_scale_factor=0.02);
```
这就把阈值定在2%了。
理论上，这样autovacuum能调度的更频繁一点。查询计划相对来说也要更准确点。