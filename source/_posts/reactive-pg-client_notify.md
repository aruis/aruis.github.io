---
title: reactive-pg-client实现数据库到应用程序的数据推送
date: 2018-09-15 17:53:48
categories: 程序人生
tags:
    - Vert.x
    - PostgreSQL
---
昨天聊过，依托`reactive-pg-client`可以做很多传统`JDBC`无法实现的事情，比如`PostgreSQL`的消息推送（`notify`和`listen`）。有了这种功能，我们就可以轻易实现从数据库层主向业务逻辑代码推送消息的功能。可以说，又一次为我们打开了新世界的大门。
### 先来回顾下`PostgreSQL`的`notify`和`listen`
主要参考[官方文档](http://www.postgres.cn/docs/10/sql-notify.html)，其实非常简单，核心`SQL`就两句话：
* 发消息：
    ```
    NOTIFY channel_name;
    ```
    `NOTIFY`是一个关键字，后面跟着的第一参数是，频道的名字，这个是用户随便定义的，只要之后跟`LISTEN`的保持一致即可。`NOTIFY`还有第二个可选参数，就是消息内容，类型也必须是字符串，并且长度限制在`8000`字节。
    `NOTIFY channel_name message_body`还有一种等效的写法，也就是
    ```
    SELECT pg_notify(channel_name，message_body）
    ```
    后者的好处是可以用到`SQL`的预编译特性。
* 收消息：
    ```
    LISTEN channel_name;
    ```
    
这里补充几个知识点：
* `NOTIFY`/`LISTEN`这些语法不是`SQL` 标准，属于`PostgreSQL`特有的功能
* 跟`LISTEN`相反的，有`UNLISTEN`可供使用
* `NOTIFY`是广播模式，也就是所有同频道的`LISTEN`都能接收到信息
* 先`NOTIFY`，再`LISTEN`是没有效果的。这个也符合直觉。
* 想测试的话，最好借助`PostgreSQL`自带的`psql`环境，可以很容易测试，其他`SQL`客户端程序可能就没那么友好了。

### 有了`PostgreSQL`的基础知识储备，我们就可以用`reactive-pg-client`尝试一把了
直接上代码
```
 pgClient.getConnection {
            def conn = it.result()

            conn.notificationHandler({ notification ->
                println("Received ${notification.payload} on channel ${notification.channel}")
                context.assertEquals(notification.payload, message)
                context.assertEquals(notification.channel, channelName)
                async.complete()
            })

            conn.preparedQuery("LISTEN $channelName", { ar ->
                println("Subscribed to channel")

                conn.preparedQuery('''select pg_notify($1,$2)''', Tuple.of(channelName, message), {})
            })
        }
```
没什么复杂的，通过`reactive-pg-client`拿到`connection`之后，先用这个`connection`注册个`notificationHandler`用来接收消息。但是此时还不够，还必须在同`connection`上执行`LISTEN`的`SQL`语句，才能保证之前的`notificationHandler`是有效的。至于`NOTIFY`语句并不要求非要用这个`connection`，这也是符合我们业务需求的。
以上完整代码，可以在我的[github](https://github.com/aruis/studyvertx/blob/master/src/test/groovy/com/aruistar/studyvertx/ReactivePostgresClientTest.groovy)中找到。
上面我是在当前代码中，手动`NOTIFY`的，我们完全可以吧`NOTIFY`放在数据库的触发器、或者数据库定时任务中，就可以实现数据库到应用程序的数据推送了。另外经测试，`NOTIFY`与`LISTEN`两个语法在`JDBC`环境下也是可以执行成功的（`SQL`不会报错），只是`LISTEN`不会有任何效果就是了。

明天，我将演示基于`reactive-pg-client`的数据库事务demo。