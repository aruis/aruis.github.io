---
title: PostgreSQL中的distinct on
date: 2018-08-09 08:24:27
categories: 程序人生
tags:
    - PostgreSQL
---
想象有这么一张表，存放若干学生不同课程的考试成绩，需求是，找出每门课程中，成绩最好的学生。原始表大概如下：

```
+--------------------------------------+-----------------+----------------+-----------+
| id                                   | v_studentname   | v_coursename   | i_score   |
|--------------------------------------+-----------------+----------------+-----------|
| fb9fe43a-6f57-4f92-8678-139167693e72 | 张三            | 语文           | 91        |
| 0898afd7-4253-496e-9190-36fae14bddf2 | 张三            | 数学           | 77        |
| 2a1f810d-55ee-42cc-ac17-ee970154cecb | 张三            | 英语           | 90        |
| fa975cd4-af5f-49d0-a89e-6f88665629eb | 李四            | 语文           | 88        |
| 8833a07c-de8d-4b15-aa34-d9340c2e82c3 | 李四            | 数学           | 87        |
| 1a9dfbdf-7c44-45d7-8141-590203aa26a9 | 李四            | 英语           | 89        |
| cd3ec937-0ab9-4745-ad1a-c9f85839eaeb | 王五            | 语文           | 89        |
| 431b7ccd-3a25-4ff4-9cc2-2e9f111f5c06 | 王五            | 数学           | 91        |
| 8021e41d-09f5-49fb-a9e2-750d14bbff50 | 王五            | 英语           | 79        |
+--------------------------------------+-----------------+----------------+-----------+
```
## 传统方案
传统的方法，要找每门课程成绩最好的学生的话，需要借助聚合函数，而且要几步操作：
1. 找到每门课程的最好成绩
    ```
select v_coursename, max(i_score)
from achievement
group by v_coursename;
    ```
2. 再跟`achievement`表`join`，才能找到该课程最高分对应的学生姓名。即使在有`CTE`支持的情况下，实现起来也很复杂
    ```
with cte as (select v_coursename, max(i_score) as i_score from achievement group by v_coursename)
select cte.*, achievement.v_studentname
from cte
       join achievement on cte.i_score = achievement.i_score and cte.v_coursename = achievement.v_coursename
order by cte.v_coursename;
    ```
    要是没有`CTE`的支持，估计用纯数据库实现，就捉襟见肘了。
    
## 使用distinct on的方案

```
select distinct on (v_coursename) * 
from achievement
order by v_coursename, i_score desc;
```
    
一句话解决，是不是很开森。

#### 原理回顾
`DISTINCT ON`是将结果集按指定字段值的去重，具体实现方法是先对结果集按照`DISTINCT ON`指定的字段进行排序，然后筛选出每个字段第一次出现时所在的记录，其余的记录都剔除。
`ON`修饰符支持多列，运算时会基于多列的总体唯一性进行去重操作。同时查询语句必须要有`ORDER BY`，并且要保证排序字段从左至右的的顺序，应该是跟`DISTINCT ON`命中字段顺序相符合，当然`ORDER BY`可以追加更多的字段。