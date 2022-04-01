---
title: PostGIS 初体验
date: 2022-03-25 14:17:43
categories: 程序人生
tags:
    - PostgreSQL
---

## 一 安装

`PostGIS`的安装及本是无障碍的。不管是`Red Hat`系的`YUM`还是`Ubuntu`系的`APT`都可以轻松安装。以`Ubuntu`为例的话，只需要轻松执行下面的明令即可：

```shell
apt-get -y install postgresql-14-postgis-3
```

这里注意上面的`14`对应的是本机已安装`PostgreSQL`的主版本号。因为`PostGIS`是作为插件存在的，所以最好先装好一个可用的`PostgreSQL`服务，然后再安装对应版本的插件。

## 二 启用插件

对`PostgreSQL`来说，一个好的实践是把安装的插件，单独控制在一个`schema`里面。因为插件往往会带来各种各样的函数或者表，我们应该把这些非自己维护的资源与自己的业务表区分开。所以建议执行下面两行`SQL`语句，以安装（启用）插件。

```sql
CREATE SCHEMA extensions;
CREATE EXTENSION postgis SCHEMA extensions;
```

## 三 简单测试

一切顺利的话，我们就可以测试`PostGIS`的基本功能了：

```sql
-- 查询北京到南京的距离，单位米
select extensions.st_distance(extensions.st_point( 116.2,39.56), extensions.st_point( 118.78,32.04), true);
```

因为我们把`PostGIS`的相关函数等资源都装在可`schema`下，所以调用函数的时候，就需要加上`schema`的名字了。这里稍微有点麻烦。所以需要修正下数据库的`search_path`。

```sql
alter database postgres set search_path = public,extensions;
set search_path = public,extensions;
```

第一句的`postgres`对应你在用的数据库名，所以要根据实际情况做些调整。同时，第一句的设置，只有在下次客户端连接到数据库的时候才会启用，如果想要当前执行`SQL`的上下文环境即时生效，所以才有了第二句。

一切设置完毕后，我们就可以直接这样执行了：

```sql
select st_distance(st_point( 116.2,39.56), st_point( 118.78,32.04), true);
```