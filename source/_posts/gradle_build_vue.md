---
title: 用Gradle打包Vue前端程序
date: 2018-07-17 06:26:30
categories: 程序人生
tags: 
    - 前端
    - vue
    - gradle
---
我这边打包和发布程序，属于重度依赖Gradle的状态。所以纵容前端程序游离在这个体系外，不利于团队的整体协作。于是有了这篇文章。
其实很简单，首先我们需要一个以Gradle为基石的项目，把前后端项目组织成这样
```
myproject
├── build.gradle
├── frontend
│   ├── build
│   ├── index.html
│   ├── node_modules
│   ├── package.json
│   └── src
├── javaweb
│   ├── build
│   ├── build.gradle
│   ├── out
│   └── src
└── settings.gradle
```
其中`frontend`文件夹就是`vue`项目的存放路径，我们先在此文件夹中，追加文件`build.gradle`，放至在package.json隔壁。填上很简单的内容：
```
plugins {
  id "com.palantir.npm-run" version "0.5.0"
}
```
代表这个项目要用到gradle-npm-run的插件
然后修改settings.gradle，追加一行
`include 'frontend'`，这样gradle就能顺利识别vue前端项目了，并且依靠插件，我们获得了这几个task
![-w148](/media/15317809896046.jpg)
其中`build`就可以实现通过`gradle frontend:build`的命令，实现gradle对vue项目的打包了。
明天，我将继续讲解，如何用gardle实现像Linux服务器，敬请期待。
