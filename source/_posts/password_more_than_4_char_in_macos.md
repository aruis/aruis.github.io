---
title: 突破mac系统要求密码不能小于4个字符的限制
date: 2018-12-08 17:47:00
categories: 实用技巧
tags: 
 - Mac
---
1. 在终端执行`pwpolicy getaccountpolicies > temp.xml`
2. 编辑temp.xml文件，例如`vim temp.xml`
3. 删除第一行`Getting global account policies`文字，保证这个文件以`<?xml`开头
4. 找到
    ```
    <string>policyAttributePassword matches '^$|.{4,}+'</string>
    ```
    替换成
    
    ```
    <string>policyAttributePassword matches '^$|.{1,}+'</string>
    ```
    保存
5. `sudo pwpolicy setaccountpolicies temp.xml`
6. 此时就可以设置最少字符长度为1个密码了