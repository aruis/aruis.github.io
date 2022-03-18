---
title: Javalin又一个小而美的Java Web框架
date: 2018-08-24 09:31:15
categories: 程序人生
tags:
    - WEB
    - Kotlin
---
`Javalin`是`JVM`平台下一个上手极为容易的`Web`框架。有这么几个显著的特点：
1. 简单便捷，不论是概念上还是真正上手开发，给人的感觉就是轻松写意
2. 灵活，可以兼容同步和异步两种编程思路
3. 小，即使是打成一个可以独立运行的`fat-jar`，大小才`4～5M`，就算把常用的`log`、`jdbc`等常用库放进去，估计也到不了`10M`

官方支持的语言是`Java`和`Kotlin`，当然还有跟`Java`无缝兼容的`Groovy`，这个相当于买一赠一了。下面我们直接看一下`Kotlin`简单demo
```java
package con.aruistar.studyjavalin

import io.javalin.Javalin

data class User(val name: String, val id: Int)

fun main(args: Array<String>) {
    val app = Javalin.create().start(7000)

    val map = hashMapOf<String, Int>()
    map.put("one", 1)
    map.put("two", 2)

    app.get("/") { ctx -> ctx.result("Hello World") }
    app.get("/json") { ctx -> ctx.json(User("Alex", 1)) }
    app.get("/json/map") { ctx -> ctx.json(map) }
    app.after { ctx ->
        println("log")
        println(ctx.resultString())
    }
}
```
有没有一种简单到没朋友的感觉，我想这段代码我不多解释，大家也都看得懂。其他特性还有很多，这里我从文档上摘录几个比较实用的：
```java
get("/hello/*/and/*", ctx -> {
    ctx.result("Hello: " + ctx.splat(0) + " and " + ctx.splat(1));
});
```
```java
app.routes(() -> {
    path("users", () -> {
        get(UserController::getAllUsers);
        post(UserController::createUser);
        path(":id", () -> {
            get(UserController::getUser);
            patch(UserController::updateUser);
            delete(UserController::deleteUser);
        });
    });
});
```
```java
app.exception(NullPointerException.class, (e, ctx) -> {
    // handle nullpointers here
});

```
```java
app.error(404, ctx -> {
    ctx.result("Generic 404 message")
});
```
更多功能请查看官方文档[javalin](https://javalin.io/documentation)

### 最后说一下我个人对这个框架的一些看法
如果你是个`Java`程序员，还从来没有用过`Spring`以外的`WEB`框架，那我推荐你试试`Javalin`；但如果你是站在公司立场，要为下一个项目做技术选型的话，我更建议你用经受过多年市场考验的[Vert.x](http://vertx.io)。
如果你的项目中已经用过[SparkJava](http://sparkjava.com)了，那我觉得`Javalin`应该也在你的备选技术清单里。
如果你是`node.js`程序员，想涉猎一下`Java`方面的开发，`Javalin`可能是个不错的尝试，因为从中你能找到`koa`的影子，我想你会觉得无比亲切。

---
本文demo已上传至https://github.com/aruis/studyjavalin