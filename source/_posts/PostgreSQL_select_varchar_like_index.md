---
title: PostgreSQL中varchar类型支持like查询索引
date: 2018-08-01 08:42:03
categories: 程序人生
tags:
    - PostgreSQL
---
之前在使用`PostgreSQL`中的`varchar`类型时想当然的以为用最基本的索引创建语句创建的索引，就支持`like`查询的。
类似这句：
```
CREATE INDEX log_action_v_uri_index ON log_action (v_uri);
```
但是看过`explain`才知道，单凭这样的索引，在`like`搜索的时候，仍然是顺序全表扫描，如图：
![](/media/15330845808519.jpg)
后来google了一下解决方案，原来是创建索引的时候追加一下参数，应该这么创建
```
CREATE INDEX log_action_v_uri_index ON log_action (v_uri varchar_pattern_ops);
```
然后对同样的语句再次`explain`结果如下：
![](/media/15330848868005.jpg)
可以看到，已经命中索引了。
这其中涉及到一个重要的知识点，就是`PostgreSQL`的索引*操作符类* ，其标准语法为
```
CREATE INDEX name ON table (column opclass [sort options] [, ...]);
```
这个`opclass`可以针对不同的数据类型和查询方式提供多种多样的方案，
![](/media/15330861199315.jpg)
可以看到光`btree`索引有关的操作符类就非常多，有些`is_default`是`true`也就是正好列类型吻合的话，这个操作符类就不要特意说明了。而`varchar`或者`text`的默认用的是`text_ops`，它支持的查询方式仅有
![](/media/15330876281185.jpg)
所以想用模糊查询的话，还是得用`text_pattern_ops`或者`varchar_pattern_ops`(这哥俩是一个东西)，它支持正则模糊匹配，也支持`like %`的方式
![](/media/15330878032471.jpg)

* 更多信息，可以查看官方文档[操作符类和操作符族](http://www.postgres.cn/docs/9.6/indexes-opclass.html)