---
title: 编写跨操作系统Java代码时动态或许文件分隔符
date: 2018-07-26 06:20:45
categories: 程序人生
tags:
    - Java
---
### 大概有以下几种思路
1. `File.separator`系统相关的默认名称分隔符，为方便起见，表示为字符串。 该字符串包含单个字符，即separatorChar。
2. `FileSystems.getDefault().getSeparator()`返回名称分隔符，表示为字符串。
名称分隔符用于分隔路径字符串中的名称。 实现可能支持多个名称分隔符，在这种情况下，此方法返回特定于实现的默认名称分隔符。 通过调用toString（）方法创建路径字符串时使用此分隔符。
对于默认提供程序，此方法返回与java.io.File.separator相同的分隔符。
3. `System.getProperty("file.separator")`

正常情况下，选择第一种，就ok了。第二种是在Java7时代追加的，功能更为强大。第三种也不错，因为额外提供通过`-Dfile.separator=`参数来指定的特性。