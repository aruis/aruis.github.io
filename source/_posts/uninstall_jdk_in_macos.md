---
title: 完整卸载MacOS里的JDK
date: 2018-11-28 08:18:01
categories: 工具技巧
tags:
    - Java
---
首先，所有的资料在[java官方网站](https://www.java.com/en/download/help/mac_uninstall_java.xml)都有提供，我这里做个搬运工，只罗列一下重点。
1. 下载[工具](https://javadl-esd-secure.oracle.com/update/jut/JavaUninstallTool.dmg)，并运行之
2. 打开终端，执行命令移除相关文件夹
    * sudo rm -fr /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin 
    * sudo rm -fr /Library/PreferencePanes/JavaControlPanel.prefPane 
    * sudo rm -fr ~/Library/Application\ Support/Oracle/Java
