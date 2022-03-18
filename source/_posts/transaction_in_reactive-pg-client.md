---
title: reactive-pg-client中的数据库事务
date: 2018-09-16 07:50:39
categories: 程序人生
tags:
    - Vert.x
    - PostgreSQL
---
使用`reactive-pg-client`中的数据库事务有两种写法，一种是通过`PgConnection`的`begin`方法，开启一段数据库事务；另一种是通过`PgPool`的`begin`方法。
先来看第一种：
```
pool.getConnection(res -> {
  if (res.succeeded()) {

    // Transaction must use a connection
    PgConnection conn = res.result();

    // Begin the transaction
    PgTransaction tx = conn.begin();

    // Various statements
    conn.query("INSERT INTO Users (first_name,last_name) VALUES ('Julien','Viet')", ar -> {});
    conn.query("INSERT INTO Users (first_name,last_name) VALUES ('Emad','Alblueshi')", ar -> {});

    // Commit the transaction
    tx.commit(ar -> {
      if (ar.succeeded()) {
        System.out.println("Transaction succeeded");
      } else {
        System.out.println("Transaction failed " + ar.cause().getMessage());
      }

      // Return the connection to the pool
      conn.close();
    });
  }
});
```
再看第二种：
```
pool.begin(res -> {
  if (res.succeeded()) {

    // Get the transaction
    PgTransaction tx = res.result();

    // Various statements
    tx.query("INSERT INTO Users (first_name,last_name) VALUES ('Julien','Viet')", ar -> {});
    tx.query("INSERT INTO Users (first_name,last_name) VALUES ('Emad','Alblueshi')", ar -> {});

    // Commit the transaction and return the connection to the pool
    tx.commit(ar -> {
      if (ar.succeeded()) {
        System.out.println("Transaction succeeded");
      } else {
        System.out.println("Transaction failed " + ar.cause().getMessage());
      }
    });
  }
});
```
可以看到第二种写法更为精简，`Handler`内部直接可以拿到`PgTransaction`，不需要像第一种那样使用`PgConnection`的`begin`才能得到`PgTransaction`，并且因为第二种不存在直接操纵`PgConnection`，自然也省略了手动`close`的步骤。
经过我自己测试，有两点需要注意一下：
* `PgTransaction`可以通过`abortHandler`方法，捕捉到事务失败（如果失败的话）的事件。
* 在使用事务的过程中，想通过`preparedQuery`使用预编译的`SQL`未能成功，目前原因未知。

详细的演示代码，可以在我的[github](https://github.com/aruis/studyvertx/blob/master/src/test/groovy/com/aruistar/studyvertx/ReactivePostgresClientTest.groovy)中找到。
