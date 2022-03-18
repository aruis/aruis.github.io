---
title: PostgreSQL中的行级权限/数据权限/行安全策略
date: 2018-09-28 10:10:07
categories: 程序人生
tags:
    - PostgreSQL
---
`PostgreSQL`中是可以针对不同用户，按行过滤数据的，这其实跟管理信息系统里经常提到的`数据权限`是干一个事情。但是由数据库自身提供这个功能，听起来还是很强大的。真要动手操作起来也并不复杂，主要有这么几个要点：
### 1. 开启行级权限
```
ALTER TABLE 目标表名 ENABLE ROW LEVEL SECURITY;
```
开启行级权限后，除了表的拥有者，其他用户都是默认否定权限，也就是不可能从表中查询到数据，当然其他`DML`操作也不行。

### 2. 创建策略
```
CREATE POLICY 起个策略名 ON 目标表名 TO 目标角色名
    USING (一个能返回boolean的表达式);
```
当表达式返回`true`时，该行就是可见的，反之是隐藏的。如果是update或delete一个不符合策略的数据，不会报错，只会略过。
### 3. 精确到命令的策略
策略是可以精确到`SELECT`、`INSERT`、`UPDATE`以及`DELETE`的，写在`TO`之前，用`FOR`打头，像这样
```
CREATE POLICY 起个策略名 ON 目标表名 FOR 命令名(比如select) TO 目标角色名
    USING (一个能返回boolean的表达式);
```
如果不写命令的话，默认是`ALL`

### 4. 由于支持命令级别的策略，实际的复杂度大大上升了
听我细细讲来：
* `select`、`delete`是面对现存数据的命令
* `insert`是面对新产生数据的命令
* `update`跨界，它既要面对现存数据，又要产生新数据

这么看，上面只有一个`USING`表达式的思路就说不通了，在只有一个`USING`的情况下，它到底是描述`数据的可见性`问题还是描述`数据的合规性`问题呢？这就引出了`WITH CHECK`表达式。`USING`负责判断数据的可见性，`WITH CHECK`符合判断数据的合规性。这里拿`update`举例子比较合适
```
create policy my_policy on users
  for update
  to joe
  using (user_name = current_user)
  with check (age > 18)
```
这里`USING`的表达式限制了只有字段`user_name`等于当前用户名的数据才可以被编辑（current_user是系统内置变量）；而编辑后的结果，只有`age`大于18才可以成功存储。用过数据库约束的，现在肯定明白了，`with check`就是数据库约束嘛，只是这个约束不是面向表的，而是面向角色（用户）的。
这也就解释了当用户尝试`update`它没权限`update`的数据，并不会报错，而`update`的数据不满足`check`时，会收到一个错：
```
ERROR:  new row violates row-level security policy for table "users"
```

### 5. 如果策略不止一个，那它们之间的关系是`OR`？还是`AND`？
答案是默认情况是`OR`，这个叫`宽松策略`。如果引入`RESTRICTIVE`关键字就成为`限制性策略`。其用法如下
```
CREATE POLICY  users_managers ON users as RESTRICTIVE  for select  TO joe  using(first_name='a');
```
所有`限制性策略`都是`AND`链接的，在跟`宽松策略`组合起来使用时，会这么一个情况
```
(宽松策略A or 宽松策略B) and (限制性策略C and 限制性策略D)
```
不过还有一点要注意，**如果只有限制性策略时不行的，必须先有一个宽松策略才行，这个可以理解成`限制性策略`必须得跟别人`and`，如果不提供`宽松策略`的话，就是`and (空)`，所以结果只能是空**。

### 6. 官方文档
`CREATE POLICY`的详细语法解析[在此](http://www.postgres.cn/docs/10/sql-createpolicy.html)
`行级权限（行安全性策略）`的整体解释和示例代码[在此](http://www.postgres.cn/docs/10/ddl-rowsecurity.html)

