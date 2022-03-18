---
title: Gradle插件下载不下来的解决方案
date: 2018-09-04 15:06:51
categories: 程序人生
tags:
    - gradle
---
字符乱码和互联网不互联，是困扰中国程序员的两大问题。我就在使用`gradle`的时候，会遇到官方插件下载不下来的情况。
比如使用`org.hidetake.ssh`插件时，如果按照文档所述，直接这样写
```
plugins {
  id 'org.hidetake.ssh' version '2.9.0'
}
```
有时候就会遇到网络问题，因为此时要去访问`gradle.org`官网去申请插件，而不知道什么时候这个网络就不通了。
此时我们可以通过细化`buildscript`的`repositories`来解决问题，也就是在`plugins`之前，增加`buildscript`的配置内容，整体代码如下
```
buildscript {
    repositories {
        maven {
            url "http://maven.aliyun.com/nexus/content/groups/public"
        }
        jcenter()
    }
}

plugins {
    id "org.hidetake.ssh" version "2.9.0"
}
```
*这里我们通过使用阿里云的`maven`服务器地址，理论上还能在国内获得更快的资源访问速度。*