---
title: nginx中使用rewrite重定向
date: 2018-09-20 15:31:30
categories: 程序人生
tags:
    - nginx
---
`nginx`中配置重定向，虽然有下面这种写法
```
return 301 https://www.yourdomain.com$request_uri;
```
但是仍不及`rewrite`好用强大。其基本语法是
```
Syntax:	rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
```
举个例子
```
rewrite ^/(.*)$ https://www.qxnaqy.com permanent;
```
`rewrite`后面紧跟的是正则表达式，用来匹配url。而`replacement`可以是`http`开头的绝对路径，就会触发重定向。如果不是绝对路径，则默认是触发重写。
重定向与重写的区别是，前者是浏览器有感知的，通过`302`、`301`通知浏览器url资源发生了变化，由浏览器再次发起请求，访问目标路径；后者是浏览器无感知的，由`nginx`延续接下来已改变的请求。
但是如果`flag`指定了`redirect`或者`permanent`时，一定是触发的重定向。
文档中关于`flag`的解释如下
```
last
stops processing the current set of ngx_http_rewrite_module directives and starts a search for a new location matching the changed URI;
break
stops processing the current set of ngx_http_rewrite_module directives as with the break directive;
redirect
returns a temporary redirect with the 302 code; used if a replacement string does not start with “http://”, “https://”, or “$scheme”;
permanent
returns a permanent redirect with the 301 code.
```
也就是说，如果想做`301`重定向，务必使用`permanent`，如果想做`302`，根据`replacement`的内容来决定是否使用`redirect`。
`last`的意思是，命中更改的`uri`之后，接着拿新的`uri`尝试匹配。这就会出现一种情况，当新的`uri`规则又满足之前匹配模式，就会进入一种死循环状态，所以就引入了`break`。可以参考文档中的这段
```
The full redirect URL is formed according to the request scheme ($scheme) and the server_name_in_redirect and port_in_redirect directives.

Example:

server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}
But if these directives are put inside the “/download/” location, the last flag should be replaced by break, or otherwise nginx will make 10 cycles and return the 500 error:

location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}
```



