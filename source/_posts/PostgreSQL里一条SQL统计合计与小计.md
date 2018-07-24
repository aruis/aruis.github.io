---
title: PostgreSQL里一条SQL统计合计与小计
date: 2018-07-22 18:30:31
categories: 程序人生
tags:
    - PostgreSQL
---
```
SELECT
  CASE WHEN GROUPING(student) = 1
    THEN '合计'
  ELSE student END,
  CASE WHEN  GROUPING(student) <> 1 and GROUPING(course) = 1
    THEN '小计'
  ELSE course END,
  sum(score.score)
FROM score
GROUP BY ROLLUP (student, course)
ORDER BY GROUPING(student) DESC ,student DESC, GROUPING(course) DESC ,course DESC;
```