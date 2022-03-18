---
title: ubuntu里查看软件包信息
date: 2018-09-05 08:49:48
categories: 程序人生
tags:
    - ubuntu
---

以`postgresql-10-plv8`这个软件包为例，需要知道软件包的一些必要信息，比如版本、项目主页、大小、简单说明之类的，可以用如下命令查看：
```
apt show postgresql-10-plv8
```
对于早期版本的ubuntu（<14.04），可以使用这个命令
```
apt-cache show postgresql-10-plv8
```
另外一种方式也是极好的，就是利用`aptitude`，前提是你先安装了它，那么就可以这么用：
```
aptitude show postgresql-10-plv8
```
`aptitude`比前者强的地方是它能告诉你该软件包是否已经安装

