---
title: 性能封神的数据库访问库reactive-pg-client初探
date: 2018-09-14 09:29:27
categories: 程序人生
tags:
    - Vert.x
    - PostgreSQL
---
不知道有没有人关注过TechEmpower的`Web Framework Benchmarks`（https://www.techempower.com/benchmarks/ ），一种把各种`Web`后端开发技术都罗列起来，在同样的机器下跑最简单的业务逻辑，来对比各技术性能的竞赛。参与跑分的技术代码，都是开源的，甚至很多代码就是由`Web`技术本身的作者维护的。
虽说，跑分不能代表性能，性能更不能代表技术的优劣。但是有着么一个性能维度的粗浅比较，还是能带给我们不一样的收获。
比如，在`2018-06-06`的最新一场比拼中，`Vert.x`搭配`PostgreSQL`的组合，在部分比试时表现十分抢眼，性能一骑绝尘且大幅领先第二名。被它踩在脚下摩擦的对手，不乏我们熟知的技术方案，比如`go`、`nodejs`、`spring`、`php`，还有`mysql`、`mongodb`等数据库。（这里我无意引战，有兴趣的朋友可以查看具体[跑分结果](https://www.techempower.com/benchmarks/#section=data-r16&hw=ph&test=db)及[相关代码](https://github.com/TechEmpower/FrameworkBenchmarks/tree/round16)）
查看具体跑分[代码](https://github.com/TechEmpower/FrameworkBenchmarks/blob/724a773096f403d149cfaf59b8257465bbf70103/frameworks/Java/vertx/src/main/java/vertx/App.java)可知，`Vert.x`之所以能位居榜首，与其使用的数据库客户端是密不可分的。作为一套基于`Java`的技术解决方案，`Vert.x`没有使用`Java`程序员所熟知的`JDBC`，而是使用了一种叫[reactive-pg-client](https://github.com/reactiverse/reactive-pg-client)的技术。
`reactive-pg-client`与`JDBC`最大的区别就是前者是针对`PostgreSQL`数据库单独开发的，利用了`PostgreSQL`异步特性，最大限度了挖掘了数据库的访问性能。
现在我们就来简单尝试下`reactive-pg-client`，首先添加依赖
```
compile 'io.reactiverse:reactive-pg-client:0.10.3'
```
准备`PgPoolOptions`
```
PgPoolOptions options = new PgPoolOptions()
  .setPort(5432)
  .setHost("the-host")
  .setDatabase("the-db")
  .setUser("user")
  .setPassword("secret")
  .setMaxSize(5);
```
接下来就可以用了
```
// Create the client pool
PgPool client = PgClient.pool(options);

// A simple query
client.query("SELECT * FROM users WHERE id='julien'", ar -> {
  if (ar.succeeded()) {
    PgRowSet result = ar.result();
    System.out.println("Got " + result.size() + " rows ");
  } else {
    System.out.println("Failure: " + ar.cause().getMessage());
  }

  // Now close the pool
  client.close();
});
```
典型的异步代码编写风格，正是由于该库先天异步的特性，我们可以用一个线程，就能控制多个数据库链接（The client is reactive and non blocking, allowing to handle many database connections with a single thread.），籍此获得更好的计算机资源利用率，从而提高性能。更多信息请查看官方文档[reactive-pg-client](https://reactiverse.io/reactive-pg-client/guide/java/)

明天，我将继续深入`reactive-pg-client`，带你尝试`PostgreSQL`独有的`NOTIFY`特性，实现从数据库端到业务程序段的数据主动推送。