---
title: groovy中的正则使用
date: 2018-08-21 10:24:14
categories: 程序人生
tags:
    - groovy
---
#### 判断是否与正则匹配
```
def res = "522300000000" ==~ /\d*0{8}$/
// res is true
```

#### 从字符串中找到匹配的内容
```
def res = "hello110,world" =~ /\d+/
println(res[0])
//res[0] is 110
```