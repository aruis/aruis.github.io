---
title: Gradle中的sourceCompatibility与targetCompatibility
date: 2018-08-27 11:45:57
categories: 程序人生
tags:
    - gradle
---
`Gradle`中有两个属性，我也是一知半解。今天正好要练习下`Java10`，那就先把这两个属性的问题测试透了再说。那就是`sourceCompatibility`和`targetCompatibility`。
简单的说，`sourceCompatibility`属性跟编译环境有关，而`targetCompatibility`属性跟运行环境有关。
至少有这么几个原则，是不能违背的：
1. `sourceCompatibility`关系到你使用到的`Java`语法特性及库
2. `sourceCompatibility`不能比`targetCompatibility`大
3. `targetCompatibility`不能比目标客户端运行环境的`JavaVersion`大
4. `targetCompatibility`不能比当前`Gradle`使用的`JavaVersion`大

总结起来就是这样
```
代码用的语言特性对应的JavaVersion 
≦ sourceCompatibility 
≦ targetCompatibility 
≦ Gradle使用的JavaVersion 
≦ 客户端环境的JavaVersion
```
