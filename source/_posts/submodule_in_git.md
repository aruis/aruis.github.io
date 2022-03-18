---
title: 在git中使用submodule
date: 2018-09-23 07:53:15
categories: 程序人生
tags:
    - git
---
如果`clone`一个项目后，发现其中有`.gitmodules`文件，就说明这个项目是包含子项目的。这个文件的产生，是由于在一个`git`项目内，执行命令
```
git submodule add a_git_project_path rack
```
最后一个参数是创建文件夹的名字，这个跟执行`git clone`时的用法一样。
当`clone`项目后发现存在`.gitmodules`文件后。可以注意观察下，相关的submodule对应的文件夹里，是没有内容的。此时需要通过
```
git submodule init
git submodule update
```
分别初始化submodule和获取其数据。
有了`submodule`的存在，我们可以很方便的拆分一个大项目到若干小项目，方便多人合作开发。
下一次，我将就`submodule`的嵌套性展开测试。