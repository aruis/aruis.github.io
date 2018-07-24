---
title: IntelliJ IDEA在Winodws下Gradle乱码问题
date: 2018-07-23 08:19:25
categories: 程序人生
tags:
    - gradle
---
恐怕没有哪个中国程序员没被字符编码的问题坑过吧。本以为把能设置字符集的地方，都设置成`UTF-8`就不会踩坑。可是现实是残酷的。
比如在中文windows系统环境下，如果使用IntelliJ IDEA开发工具，同时跑gradle项目，那就要小心了。
需要在`File | Settings | Build, Execution, Deployment | Gradle`下，找到`Gradle VM options`，然后填入配置`-Dfile.encoding=UTF-8`
还有一种方法，可以通过修改gradle.bat这个文件来实现，改文件通常存放于，`GRADLE_HOME`下的`bin`目录，找到`set DEFAULT_JVM_OPTS=`修改为`set DEFAULT_JVM_OPTS="-Dfile.encoding=UTF-8"`即可。