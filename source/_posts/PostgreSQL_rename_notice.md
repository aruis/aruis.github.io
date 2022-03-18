---
title: PostgreSQL中对用户重命名需要注意的
date: 2018-09-25 11:05:17
categories: 程序人生
tags:
    - PostgreSQL
---
在`PostgreSQL`中，如果需要对数据库用户重命名，其实很简单，就是
```
ALTER USER name RENAME TO new_name
```
但是你会收到一个提示
```
NOTICE:  MD5 password cleared because of role rename
```
这里面传递了两个信息：
1. 改用户名，就是改角色名，因为在`PostgreSQL`中，用户等价于角色，唯一的区别就是，角色分可以登陆和不可以登陆的，而用户就是能登陆的角色。
2. 改了用户名，密码就丢了。

今天主要讨论的，就是上面第二点问题。为啥**改个用户名，密码就给我干掉了？**，这很违背常识。
但其实是有原因的：
* 首先，`PostgreSQL`中存放的密码，是MD5加密的，当然是不可逆的，这个大家能理解。
    这个通过这条`SQL`能够查到
    ```
    SELECT rolname,rolpassword FROM pg_authid;
    ```
* 但是`rolpassword`并不是`md5(password)`直接得出的，里面存的其实是
    ```
    "md5"+md5(username+password)
    ```
    这么一个情况，也就是说，如果你设置`joe`用户的密码是`xyz`的话，实际上`rolpassword`列存的值，相当于
    ```
    select 'md5'||md5('joe'||'xyz');
    ```
* 这就解释了，为什么用户名改了，密码就丢了。因为用户名本就是密码的一部分，如果加密过后的值不跟着改变的话，新的用户名沿用固定的算法，是无法登陆的。而`PostgreSQL`永远也不会知道曾经的密码是多少，所以不能自动帮用户更新`rolpassword`，这就只能靠用户自己重新设置一次密码了。
* 说白了，`PostgreSQL`也是好心，她是有苦衷的。
