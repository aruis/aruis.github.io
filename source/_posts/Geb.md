---
title: JVM下最好用的前端自动化测试工具Geb
date: 2018-08-04 13:56:28
categories: 程序人生
tags:
    - 前端
    - Geb
---
接触了`Angular`才知道前端有个端到端测试的说法，然后了解到有`WebDriver`这种神奇的存在，瞬间打开了新世纪的大门。后来几经寻觅，终于发现一个运行在`JVM`中的前端测试工具，那就是[Geb](http://gebish.org)。
来段代码：
```
import geb.Browser
 
Browser.drive {
    go "http://myapp.com/login"
     
    assert $("h1").text() == "Please Login"
     
    $("form.login").with {
        username = "admin"
        password = "password"
        login().click()
    }
     
    assert $("h1").text() == "Admin Section"
}
```
是不是很易懂，一个有`Java`与`jQuery`基础的人应该非常容易上手。
还记得那个阿里员工抢月饼事件么，估计看了新闻之后，前端程序员都会觉得0门槛，但是后端程序员，可能就会觉得自己的技术栈鞭长莫及了。有了`Geb`，我们能做的事情会更多，也会更加方便。下一篇，我将介绍如何用`Geb`来实现京东抢券。