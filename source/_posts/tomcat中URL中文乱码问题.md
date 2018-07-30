---
title: tomcat中URL中文乱码问题
date: 2018-07-29 09:31:01
categories: 程序人生
tags:
    - tomcat
    - 乱码
---
找到
```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" />
```
追加配置为
```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8"/>
```