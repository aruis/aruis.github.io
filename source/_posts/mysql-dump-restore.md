---
title: 备份mysql整库并还原到另外一处
date: 2018-10-08 16:56:17
categories: 程序人生
tags:
    - MySQL
---
### 全库
```sh
mysqldump -uXXX -pXXX --host=XXX --port=3306 --all-databases --triggers --routines --events --add-drop-database  --skip-add-locks -C| mysql -uXXX -pXXX --host=XXX --port=3306 test
```

### 制定数据库
```shell
mysqldump -uXXX -pXXX --host=XXX --port=3306 --databases db1 db2 --triggers --routines --events --add-drop-database  --skip-add-locks -C| mysql -uXXX -pXXX --host=XXX --port=3306 test
```