---
title: PostgreSQL中用递归CTE获取一张表的所有祖先表
date: 2018-08-15 10:18:00
categories: 程序人生
tags:
    - PostgreSQL
---
今天介绍两个知识点：
1. PostgreSQL的表继承
2. PostgreSQL中的可递归CTE

### 表继承
表继承是PostgreSQL独有的功能，就是在`create table`的时候可以通过`inherits`关键字指定若干个父表，如：

```sql
create table test
(
  id       varchar default uuid_generate_v4() not null
    constraint test_pkey
    primary key,
  text     varchar,
  t_create timestamp default now()
)
  inherits (basic_update);
```
继承父表，有如下几个特性：
* 父表的字段会自动加入子表中来
* 父表上的所有检查约束和非空约束都将自动被它的后代所继承， 除非使用 NO INHERIT子句明确指定。其他类型的约束（唯一、主键和外键约束）则不会被继承
* 插入子表的数据，在父表中默认是可以查询到的，除非在对父表查询时，表名前加入`only`修饰符

### 递归CTE
可选的RECURSIVE修饰符将WITH从单纯的句法便利变成了一种在标准SQL中不能完成的特性。通过使用RECURSIVE，一个WITH查询可以引用它自己的输出。一个非常简单的例子是计算从1到100的整数合的查询：

```sql
WITH RECURSIVE t(n) AS (
    VALUES (1)
  UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

### 回到今天的主题，PostgreSQL中用递归CTE获取一张表的所有祖先表
这里我直接通过创建一个函数来封装这个功能，具体逻辑就在函数中，相信有前面两个知识点的铺垫，小伙伴们应该也能看懂了

```sql
create function get_parent_tables_by_oid(me integer)
  returns character varying []
language plpgsql
as $$
DECLARE parent_tables VARCHAR [];

BEGIN


  WITH RECURSIVE x AS (SELECT * FROM pg_inherits WHERE inhrelid = me

      UNION ALL

      SELECT pg_inherits.* FROM x
                                  JOIN pg_inherits ON x.inhparent = pg_inherits.inhrelid)


  SELECT array_agg(pg_stat_user_tables.relname)

      INTO parent_tables

  FROM x
         INNER JOIN pg_stat_user_tables ON x.inhparent = pg_stat_user_tables.relid;


  RETURN parent_tables;

END;



$$;
```

其中`inhrelid`是`PostgreSQL`中的一种元数据，可以理解成表的`id`，在知道表名的情况下怎么知道它的`id`呢，可以去元数据里面查，如下

```sql
SELECT relid FROM pg_stat_user_tables WHERE relname = 'a_table';
```