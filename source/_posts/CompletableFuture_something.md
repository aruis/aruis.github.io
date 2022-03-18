---
title: 再谈CompletableFuture
date: 2018-09-12 09:06:33
categories: 程序人生
tags:
    - Java
---
今天是还之前欠的一个账，当时在[Vert.x异步方法转同步](https://www.kankanzhijian.com/2018/07/21/vertx_async_to_sync/)这篇文章里，我提到其关键点是`CompletableFuture`类，今天我们就沿着当时的代码继续掰扯一下这个强大的类。
先来回顾下当时的代码
```
import io.vertx.core.Vertx

import java.util.concurrent.CompletableFuture

String sayHello() {
    CompletableFuture completableFuture = new CompletableFuture()

    Vertx vertx = Vertx.vertx()
    vertx.setTimer(1000, {
        completableFuture.complete("hello world")
    })

    return completableFuture.get()
}

println(sayHello())
```
`vertx.setTimer`是个先天异步的东西，我们让它来模拟一个异步调用，你可以想象成从网络、磁盘或者其他什么接口，获取到那么一个字符串`hello world`。这么一个过程是异步的。然后在一个非异步程序的大环境下，后续的程序要等待这么一个结果。
此时我们用到了`CompletableFuture`，并且牵扯到其中的两个方法：`complete(T value)`、`get()`。
#### `complete(T value)`
```
If not already completed, sets the value returned by get() and related methods to the given value.
```
这个很好理解，就是给`completableFuture`塞一个完成的结果，供后续的方法调用获取，最典型的就是`get()`
#### `get()`
```
Waits if necessary for this future to complete, and then returns its result.
```
这个`get()`方法源自于`Future`接口，是一个早在`Java 1.5`时代就提供的接口了。这个方法就是典型的阻塞式获取`Future`结果。放在上面的代码里，恰好能起到把`vert.x`的异步调用转换成同步的效果。但其实着不是什么好事，在`Java 8`中特意引入`CompletableFuture`就是为了解决阻塞问题，让异步发挥出更大的优势。

#### 发散一下
抛开上面有意把异步转同步不说，我们来看看如果借助`CompletableFuture`，把`vert.x`的异步跟`Java 8`的异步有效结合，尝试代码如下：
```
import io.vertx.core.Vertx

import java.util.concurrent.CompletableFuture

CompletableFuture sayHello() {
    CompletableFuture completableFuture = new CompletableFuture()

    Vertx vertx = Vertx.vertx()
    vertx.setTimer(1000, {
        completableFuture.complete("hello world")
    })

    return completableFuture
}

sayHello().whenCompleteAsync({ res, th ->
    println(res)
})
```
简单改造过后，这就是一个遵循`Java 8`中`CompletableFuture`风格的异步使用方式。关键点在于
`whenCompleteAsync(@NotNull BiConsumer<? super T, ? super Throwable> action)`
方法，同时还有
`whenComplete(@NotNull BiConsumer<? super T, ? super Throwable> action)`
方法可供使用。这两个方法最大的区别是，前者会为`action`的执行上下文准备`ForkJoinPool`线程池环境；而后者会让`action`使用之前`completableFuture.complete()`所处的线程上下文。

#### `CompletableFuture`是一个异常强大且复杂的类，本文所讲的东西不过是九牛一毛。这里推荐两个不错的帖子供大家参考：
* https://colobu.com/2016/02/29/Java-CompletableFuture/
* https://www.ibm.com/developerworks/cn/java/j-cf-of-jdk8/index.html

    其中第二篇文章涉及的代码，我已经整理了一份，你可以从这里获取[gist](https://gist.github.com/aruis/135f6e1fa678fc5024ea20db4b9b4eee)



