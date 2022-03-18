---
title: 记一个因为JDBC及PostgreSQL过于优秀而导致的坑
date: 2018-08-23 07:16:21
categories: 程序人生
tags:
    - PostgreSQL
---
假设存在这么一张表，记录全国各地大学的名称，以及所处的行政区划
```sql
CREATE TABLE 大学信息表
(
    id varchar DEFAULT uuid_generate_v4() PRIMARY KEY,
    v_name varchar,
    v_所处行政区划代码 varchar
);
```
其中，行政区划是国家标准的，也就是你身份证开头的6位，能够精确到区、县的。两位一个级别，前六位，可以笼统的概括成，省及、市级、县级，当然还有直辖市、特别行政区什么的，先不必深究，暂且简化这个逻辑就好了。
现在问题来了，如果想统计各省份大学的数量，是不是该用这条`SQL`
```sql
select left(v_所处行政区划代码, 2), count(*)
from 大学信息表
group by left(v_所处行政区划代码, 2);
```
对需求敏感的小伙伴肯定看出来了，上面的`2`在设计接口的时候应该设计成参数，万一要统计各城市的大学数量，直接传`4`不就好了。
所以在`Java`中，我们大约会写这么一段代码
```sql
PreparedStatement pstmt = conn.prepareStatement("""
select left(v_所处行政区划代码, ?), count(*) as num
from 大学信息表
group by left(v_所处行政区划代码, ?);
""");

pstmt.setInt(1, 2);
pstmt.setInt(2, 2);
```
就是用两处`?`占位，然后传入相同的参数。**可惜事与愿违，这段`SQL`一定是报错的**，会有提示
```sql
column "大学信息表.v_所处行政区划代码" must appear in the GROUP BY clause or be used in an aggregate function
```
简单的说，数据库不认为`select`中的`left(v_所处行政区划代码, ?)`跟`group by`中的`left(v_所处行政区划代码, ?)`是一个东西，所以认为我们在查询了一个没有`group by`也没有聚合函数的数据，属于`SQL`语法错误。
下面尝试分析原因：
1. 原始`SQL`在`PostgreSQL`中独立执行，完全没问题，所以肯定不是`PostgreSQL`的问题
2. 用`Python`（`psycopg2`）实现了同样的逻辑，然而没有报错，所以还得从`JDBC`+`PostgreSQL`上找原因
3. 去数据库服务器看日志，发现问题，当`JDBC`请求的时候，后台日志是
```sql
STATEMENT:
	select left(v_所处行政区划代码, $1), count(*) as num
	from 大学信息表
	group by left(v_所处行政区划代码, $2)
DETAIL:  parameters: $1 = '2', $2 = '2'
```
    而当`Python`请求时，后台日志是
```sql
select left(v_所处行政区划代码, 2), count(*) as num
		from 大学信息表
		group by left(v_所处行政区划代码, 2)
```
    是不是有一种见了鬼的感觉，

### 结论
在使用`JDBC`操作`PostgreSQL`时，`JAVA`中的`prepareStatement`会精准换算成`PostgreSQL`中的[PREPARE](http://www.postgres.cn/docs/10/sql-prepare.html)，而在`PREPARE`的时候，参数还没给出，所以`PostgreSQL`会认为`select`的`left(v_所处行政区划代码, $1)`并未参与`group by`，因而`PREPARE`报错，导致最终`SQL`执行失败。
而在`psycopg2`+`PostgreSQL`的环境中，并没有像`JDBC`那样充分利用`PostgreSQL`的`PREPARE`特性，而是在程序侧就换算好了`SQL`语句，所以反而不会报错。
*显然，是`JDBC`的设计更细致，但是却给自己挖了个坑*