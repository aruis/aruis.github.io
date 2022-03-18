---
title: MySQL日期字段同时实现新增默认值及修改默认值
date: 2018-10-15 15:47:05
categories: 程序人生
tags:
    - MySQL
---
如果说`MySQL`有什么功能是值得我留恋的，那这个功能绝对能排第一。要知道在`PostgreSQL`实现这么个修改默认值，还非得写个触发器不可，对新手来说太不友好了（具体可参考[PostgreSQL中实现更新默认值](https://www.kankanzhijian.com/2018/08/16/PostgreSQL_auto_modify_defaults_1/)）。
来看下在`MySQL`里怎么做，简单的一行
```
ALTER TABLE datalock ADD t_update  timestamp default current_timestamp on update current_timestamp COMMENT '变动时间';
```
就可以把新增默认值、修改默认值同时设置。如果只想设置修改默认值，可以更简单：
```
ALTER TABLE datalock ADD t_update  timestamp  on update current_timestamp COMMENT '变动时间';
```
