---
title: 用JavaScript在PostgreSQL中写存储过程
date: 2018-09-09 08:27:59
categories: 程序人生
tags:
    - PostgreSQL
---
首先交代一个概念，在`PostgreSQL`中，函数、存储过程都是一回事，创建语法都是`create function`。之所以本文标题使用`存储过程`的叫法，是为了方便其他数据库的使用者容易理解。
先来看看`PostgreSQL`默认支持的创建数据库函数的写法，如：
```
-- 把系统生成的uuid中的-替换成_
create function uuid()
  returns text
language sql
as $$
SELECT replace(uuid_generate_v4()::text,'-','_');
$$;
```
注意其中的`language`关键字，后面指明了该数据库函数所使用的语言，上面这个函数很简单，用纯`SQL`就能搞定，更多的时候，我们遇到的会复杂很多，就得用上过程化的SQL，也就是`plpgsql`（类似Oracle中的PL/SQL），具体可以查看官方文档[plpgsql](http://www.postgres.cn/docs/10/plpgsql.html)。
不过今天的主题不是`plpgsql`，而是`plv8`——基于`V8`引擎的在`PostgreSQL`中运行的过程化语言，其实就是用`JavaScript`在`PostgreSQL`中写数据库函数。
**那么为什么要用`Javascript`来写数据库函数呢？**在我看来，至少有下面几个好处：
1. 熟悉的配方，熟悉的味道。`JavaScript`作为`Web`世界的一等公民，其教众众多。能用`JavaScript`来实现高级的数据库开发，可以大幅拉低数据库的学习曲线，进而**降低人力成本**。
2. 获得`SQL`世界本身不具备的库函数，像这样：![](/media/15364987364827.jpg)
    还有这样：
    ![](/media/15364989242348.jpg)
    可以极大丰富数据库层面的功能实现，进而影响一个软件的架构设计。
3. 更快的性能，众所周知`Google`主推的`V8`引擎是业界公认的顶尖性能怪兽。有它配合你的数据库使用真是如虎添翼。尤其是在业务系统的开发采用了并不是以性能为卖点的语言时（比如`Ruby`、`PHP`），把部分业务逻辑通过`JavaScript`在数据库中重构一边，说不定会有意想不到的效果。这比伤筋动骨地单纯在业务层摸索改进要容易得多。

说了这么多，怎么才能在`PostgreSQL`中用上`JavaScript`呢？
* 第一步肯定是在电脑上装好`PostgreSQL`，这个不再赘述
* `windows`用户，可以下载这里的安装包，http://www.postgresonline.com/journal/archives/379-PLV8-binaries-for-PostgreSQL-10-windows-both-32-bit-and-64-bit.html 
* `ubuntu`用户，`apt-get install postgresql-10-plv8`
* 更多安装细节，请参考官方文档：https://plv8.github.io 以及 https://pgxn.org/dist/plv8/doc/plv8.html
* 当然，如果你熟悉`docker`，可以使用我在`docker store`上分享的`lovearuis/postgres10_plv8`（[地址](https://store.docker.com/community/images/lovearuis/postgres10_plv8)），只需要这么一个命令
```
docker run --name=postgres10_plv8 -d -p 5432:5432 lovearuis/postgres10_plv8
```
    就可以在本机`5432`端口上运行一个装好`JavaScript`支持的最新版`PostgreSQL`
* 安装完必要的`plv8`包之后，还要在数据库中执行`SQL`：`CREATE EXTENSION plv8;`，才能真正解锁`plv8`的洪荒之力。

一切就绪之后，按照官方文档上举例的，先来个带`for`循环和生成`JSON`的耍耍，`SQL`如下：
```sql
CREATE FUNCTION plv8_test(keys TEXT[], vals TEXT[]) RETURNS JSON AS $$
  var o = {};
for(var i=0; i<keys.length; i++){
o[keys[i]] = vals[i];
}
return o;
$$ LANGUAGE plv8 IMMUTABLE STRICT;
```
尝试调用刚刚创建的`plv8_test`函数：
```
SELECT plv8_test(ARRAY['name', 'age'], ARRAY['Tom', '29']);
```
获得结果![](/media/15365022296404.jpg)
是不是美妙极了？☺️

### 放大招
是不是看了上面的介绍，有点心痒难耐了。🤪**其实`PostgreSQL`在内部可调用的过程化语言的支持远不仅于此。你完全可以用你心爱的`Python`、`Java`、`PHP`甚至是`R`、`Lua`从事`PostgreSQL`中的数据库函数开发。**
尤其是`Python`、`Java`，这两个语言几乎在`PostgreSQL`的`PL`环境下有最大的权限空间。想象这样一个场景：
一个别人做的项目（你没有源码，或者有源码跟没有也没啥区别），领导说要加个需求，当一个数据到达某种阀值时，要发短信给一个人（比如考试成绩低于60的时候，发个短信给学生家长）。这个时候，你不需要再打开开发环境，在别人的代码里面流离失所，久久不能自拔。你需要做的，就是打开这个项目的`PostgreSQL`（谢天谢地，它用了世界上最先进的开源数据库，尽管它的代码跟翔没什么区别），然后用`Java`在`PostgreSQL`中写个数据库函数（请求一个短信网关的http接口或者别的什么东西），最后套一个触发器（3行`SQL`）就什么都完成了。
这画面太美，我不敢想象啊，哈哈。
