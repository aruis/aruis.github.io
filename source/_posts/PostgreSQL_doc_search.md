---
title: PostgreSQL中文文档正确的搜索方式
date: 2018-09-21 13:44:54
categories: 工具技巧
tags:
    - PostgreSQL
---
今天介绍的主要是搜索引擎使用技巧，其实跟`PostgreSQL`关系不大，仅仅以对`PostgreSQL`的中文文档搜搜举个例子。
比如想了解`PostgreSQL`关于`string`类型的数据库内置函数。最直观的搜索方式，是去`google`搜`postgres string function`，这当然是一种符合直觉方式。通常情况下`google`给出的结果会不赖，如图：![](/media/15375090092852.jpg)
但是有个问题，默认搜到的内容都是英文的，这可能不利于中国的小伙伴细致地学习。但其实，我们已经有一个汉化过的文档版本，静静地躺在[PostgreSQL 中文社区](http://www.postgres.cn)，并且社区提供多个版本的汉化文档，比如：[10](http://www.postgres.cn/docs/10/)、[9.6](http://www.postgres.cn/docs/9.6/)
我们最理想的，就是期望`google`能把关键词的搜索范围限制在中文文档的地址内。其实这对搜索引擎是个再常见不过的需求了，很多小伙伴也早就知道了，那就是`site:`。
当我们希望把`postgres string function`的搜索范围仅限制在中文文档[10](http://www.postgres.cn/docs/10/)这个版本上时，仅需要在之前的搜索内容中增加`site`关键字限制范围，最终输入给`google`如下
```
postgres string function site:http://www.postgres.cn/docs/10
```
*（其实上面的postgres也可以省略了）*

来看看结果吧：![](/media/15375096548094.jpg)
现在看起来就很完美了，