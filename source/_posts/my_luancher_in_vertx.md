---
title: 分享一个Vert.x的自定义Launcher
date: 2018-08-12 07:30:14
categories: 程序人生
tags:
    - Vert.x
---
在`Vert.x`的官方example中，`mainClassName`一般是指定的`io.vertx.core.Launcher`，但是在实际应用中，如果我们也不加思索的用这个`Launcher`就会损失好多定制性，比如：
* blockedThreadCheckInterval（检查线程block定时时间间隔）
* warningExceptionTime（block多久后开始打印堆栈信息）
* maxEventLoopExecuteTime（允许`EventLoop`的最长执行时间）

这些设置都是要在`Vertx`实例化之前准备好的，只要不是Embedded应用（也就是自己调用`Vertx.vertx()`），那就只剩接管`Launcher`这条路了，下面分享一个`groovy`版的自定义`Launcher`


```
import io.vertx.core.Launcher
import io.vertx.core.VertxOptions
import org.slf4j.bridge.SLF4JBridgeHandler

class AruisLauncher extends Launcher {

    static {
        SLF4JBridgeHandler.removeHandlersForRootLogger();
        SLF4JBridgeHandler.install();
    }

    static void main(String[] args) {
        new AruisLauncher().dispatch(args)
    }

    @Override
    void beforeStartingVertx(VertxOptions options) {
        options.setWarningExceptionTime(10L * 1000 * 1000000)
        options.setBlockedThreadCheckInterval(2000)
        options.setMaxEventLoopExecuteTime(2L * 1000 * 1000000)
        options.workerPoolSize = 20
        super.beforeStartingVertx(options)
    }
}
```

在这个`Launcher`中，我还额外做了几件事：
* 用`slf4j`接管了`Vert.x`的日志
* 提供一个`main`方法，方便`IDE`开发环境启动