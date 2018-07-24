---
title: 用了docker之后性能损耗多少？
date: 2018-07-15 20:15:08
categories: 程序人生
tags: 
    - Docker
    - PostgreSQL
---
Docker的一大优势，就是拥有微乎其微的性能损耗，换来良好的硬件资源隔离效果。虽然各大厂商都在主要业务领域使用了Docker，已经从侧面表明Docker的性能损耗不是个什么问题。但是较真的同学，肯定还是想知道Docker到底有没有损耗呢，损耗多少。
这里我用PostgreSQL数据库做个简单的对比测试。分别在同一台服务器的Docker内外个各装一个10.3版本的PostgreSQL。然后执行同一个SQL。SQL如下：
```
with cte as ( select *
FROM
  (
    VALUES (uuid_generate_v4(), 'xiaoming',10, '语文'),
      (uuid_generate_v4(), 'xiaohong',12, '数学'),
      (uuid_generate_v4(), 'xiaoli',11, '英语'),
      (uuid_generate_v4(), 'xiaozhi',11, '英语'),
      (uuid_generate_v4(), 'xiaoxin',11, '英语')
  )
    AS tmp (id, name,age, fav))

select array_to_json(array_agg(row_to_json(cte)))
from cte;
```
因为不是从硬盘IO，所以这条SQL更贴近于CPU密集型的场景。压测功能是依托JMH开发的，源码已经上传至[https://github.com/aruis/somebenchmark/tree/sqlbench](https://github.com/aruis/somebenchmark/tree/sqlbench)，下面直接看结果：
![-w567](/media/15316574043348.jpg)
可以看到第一行的吞吐量，大概是第二行的95%，亦即是说，在我这个应用场景下，Docker有5%的性能损耗。
应该可以给大家做个参考。