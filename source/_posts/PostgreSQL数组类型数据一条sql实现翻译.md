---
title: PostgreSQL数组类型数据一条sql实现翻译
date: 2018-07-13 13:32:18
categories: 程序人生
tags: PostgreSQL
---
PostgreSQL的ARRAY类型是个非常实用的类型。以往在设计“多选”这种业务场景的时候，要么需要设计子表，要么弄个varchar字段，存放`1,3,5`这种逗号隔开的数据。现在有了原生支持的ARRAY类型，终于可以大胆的把多选的数据id放在这个字段里了。
接踵而至的问题是，如何一次性实现数组字段的数据翻译呢。比如实际数据是`{1,3,5}`，关联查询后，希望看到`{红,黄,蓝}`
话不多说，直接上sql
```
select distinct app_message.id,app_message.ids_at_auth_user__to,
  array_agg(auth_user.v_username) over (partition by app_message.id) as av_username_at_auth_user
from app_message
  join auth_user on auth_user.id = ANY (ids_at_auth_user__to)
```
`app_message`是个收发消息表，里面`ids_at_auth_user__to`字段是个ARRAY，存储了`auth_user`表的若干个id，代表收件箱的人（多人）
![屏幕快照 2018-04-27 11.41.45](/media/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-27%2011.41.45.png)

