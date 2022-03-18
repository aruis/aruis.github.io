---
title: PostgreSQL中的几中常见索引及应用场景
date: 2018-08-03 08:22:39
categories: 程序人生
tags:
    - PostgreSQL
---
## B-tree
`B-tree`是关系型数据库中，最常见的索引，也是`PostgreSQL`中经常采用的默认索引。主要应对场景：
*  主键
*  唯一性约束
*  等值比较
*  范围查询
*  null判断

## GiST
全称Generalized Search Tree，大白话就是通用搜索树。这是一种**有损**索引，主要应对非结构化数据，比如空间、全文检索什么的。主要应对场景：
* 几何类型数据
* 范围类型数据
* ltree类型数据
* 总之就是要用到不局限于`<、<=、=、 >=、>`这几种操作符时，要考虑`GiST`索引
* `GiST`跟`B-tree`并不矛盾，可以把`GiST`当作一种补充来用，效果更好

## GIN
全称Generalized Inverted Index，通用逆序索引。它是从`GiST`派生出来的一种索引，比`GiST`最大的优势是无损，也就是说如果要查询的数据都被索引，就可以从索引中直接获取查询结果。`GIN`索引的缺点是更新操作时，多出一个字段值复制的动作，这点不及`GiST`的速度快。另外就是它不能对大对象类型索引，比如hstore、text里面有大对象，就不适合用`GIN`了。主要应对场景：
* jsonb数据类型
* 数组数据类型
* 配合`create extension pg_trgm`对`varchar`模糊查询

## SP-GiST
SP是Space-Partitioning的简写，也就是说本索引是基于空间分区树算法的GiST索引。与`GiST`的应用场领域高度重叠，好处就是针对某些领域的特定算法，其效率要高一些。目前支持该索引的类型主要有：
* point
* box
* text
* range
* network

## Hash
PostgreSQL已将`Hash`索引列为不推荐使用状态。只能实现`=`运算相关判断。

想了解更多索引的应用范围，完全可以在`PostgreSQL`的元数据中找到答案。尝试执行这个`SQL`吧
```
SELECT am.amname AS index_method,
       opf.opfname AS opfamily_name,
       amop.amopopr::regoperator AS opfamily_operator
FROM pg_am am, pg_opfamily opf, pg_amop amop
WHERE opf.opfmethod = am.oid AND
      amop.amopfamily = opf.oid
ORDER BY index_method, opfamily_name, opfamily_operator
```