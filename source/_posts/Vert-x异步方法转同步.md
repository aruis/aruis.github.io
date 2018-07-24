---
title: Vert.x异步方法转同步
date: 2018-07-21 07:16:12
categories: 程序人生
tags:
    - Vert.x
---
以前用Vert.x的时候就有这样的疑问，如果我提供的方法是基于Vert.x异步实现的。如何被一个同步的应用调用呢，比如Spring。当时我一度以为要自己开一个线程，然后不断轮询结果，之后再返回，通过这样，把一个异步的方法，包装成同步的方法。
后来在使用`vertx-pac4j`的时候，无意中看到它源码中，也有我上面说的场景使用。就在`org.pac4j.vertx.context.session.VertxSessionStore`类的这一段：
```
 @Override
    public Session getSession(String sessionId) {
        final CompletableFuture<io.vertx.ext.web.Session> vertxSessionFuture = new CompletableFuture<>();
        sessionStore.get(sessionId, asyncResult -> {
            if (asyncResult.succeeded()) {
                vertxSessionFuture.complete(asyncResult.result());
            } else {
                vertxSessionFuture.completeExceptionally(asyncResult.cause());
            }
        });
        final CompletableFuture<Session> pac4jSessionFuture = vertxSessionFuture.thenApply(session -> {
            if (session != null) {
                return new VertxSession(session);
            } else {
                return null;
            }
        });
        try {
            return pac4jSessionFuture.get();
        } catch (InterruptedException|ExecutionException e) {
            throw new TechnicalException(e);
        }
    }
```
可以明显看到`sessionStore.get`是的常规的Vert.x异步调用。
基于这种应用方式，我尝试用`groovy`模仿写了一下，效果显著，代码如下：
```
import io.vertx.core.Vertx

import java.util.concurrent.CompletableFuture


String sayHello() {

    def future = new CompletableFuture()

    Vertx vertx = Vertx.vertx()
    vertx.setTimer(3000, {
        future.complete("hello world")
    })

    return future.thenApply({
        return it
    }).get()
}

println(sayHello())
```
上面这种写法，主要还是依赖了java1.8的`CompletableFuture`类，后续我会单独开文章讲解这个类的使用。今天就先到这里吧。