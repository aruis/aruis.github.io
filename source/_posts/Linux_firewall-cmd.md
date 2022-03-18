---
title: Linux防火墙常用命令
date: 2018-08-19 08:52:02
categories: 程序人生
tags:
    - Linux
---
1. 安装防火墙

    ```
    yum install firewalld
    ```

2. 永久放行端口

    ```
    firewall-cmd --add-port=54321/tcp --permanent
    ```
    *临时的话，把--permanent去掉*
    
3. 使配置生效

    ```
    firewall-cmd --reload
    ```
    
4. 移除放行端口
    ```
    firewall-cmd --remove-port=54321/tcp --permanent
    ```
    
5. 查看所有放行端口
    ```
    firewall-cmd --list-ports
    ```
    
这里只列出最简单的一些用法，更多高阶用法请查看官方文档：https://firewalld.org/documentation/man-pages/firewall-cmd.html    
    