---
title: PostgreSQL中使用Python编写存储过程实现科学计算
date: 2018-09-29 10:36:35
categories: 程序人生
tags:
    - PostgreSQL
---
作为写进官方文档过程语言支持，`Python`可说是`PostgreSQL`中最适合写数据库函数的了。因为`Python`是宇宙最强胶水语言，能用`Python`就意味着打开了另一个世界的大门，比如`GPU计算`、`机器学习`什么的。当然这是后话了，今天我们先来个简单的，做些与科学计算有关的的东西（这应该也很少有人在数据库上来做）。需要这样几个步骤。
### 1. 前期准备
至少要在服务上装备好`PostgreSQL 10`、`Python 3`
### 2. 安装`postgresql-plpython-10`扩展
```
apt-get install postgresql-plpython3-10
```
### 3. 在PostgreSQL中启用扩展
```
create extension "plpython3u";
```
### 4. 尝试创建一个函数
```
create function pymax(a integer, b integer)
  returns integer
language plpython3u
as $$
if a > b:
    return a
return b
$$;
```
### 5. 执行函数
```
studypg=# select pymax(2, 3);
 pymax
-------
     3
(1 row)
```
结果完美
### 6. 尝试来点复杂的
这次引用`numpy`库，用来计算`以e为底的自然数对数`，其实就是`numpy`中封装好的`log`函数啦。
```
create or replace function pylog(x float)
  returns float
language plpython3u
as $$
import numpy as np

return np.log(x)
$$;
```
执行一下，效果完美
```
studypg=# select pylog(1);
 pylog
-------
     0
(1 row)

studypg=# select pylog(0.1);
       pylog
-------------------
 -2.30258509299405
(1 row)
```

如果遇到提示
```
[38000] ERROR: ImportError: No module named 'numpy'
```
说明服务器的`numpy`模块没安装，可以通过下面的命令安装
```
pip3 install numpy
```

### 总结
至此你已经学会了，如何在`PostgreSQL`中使用`Python`，以及借助`Python`生态的力量，去解决传统关系型数据库并未涉足的领域。希望能对你有所帮助。如果你钟爱`JavaScript`，可以看我之前写的[用JavaScript在PostgreSQL中写存储过程](https://www.kankanzhijian.com/2018/09/09/plv8_in_postgresql)，也不失为一个有意思的尝试。