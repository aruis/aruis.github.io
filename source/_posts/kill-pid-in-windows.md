---
title: 在windows里杀进程，遭遇提示：没有此任务的实例在运行
date: 2022-04-11 16:14:07
categories: 程序人生
tags:
    - 运维
---

今天有被windows给坑了一把，有个几个进程死活结束不掉，还遇到提示：“没有此任务的实例在运行”，所以做个笔记。

以名称为`xxx.exe`、`PID`为`1024`为例，正常来说，杀掉一个进程命令如下：

```
taskkill /F /PID 1024
```

如果杀不掉，可能是因为有父进程影响，需要先杀父进程，而获取父进程的命令为：

```
wmic process where ProcessId=1024 get ParentProcessId
```

如果不知道子进程的`PID`，但是知道进程名字，也可以试试这个：

```
wmic process where name="xxx.exe" call terminate
```

总之找到父进程`PID`后，再用第一行命令去杀父进程才是正解。

另外补充一下，如果不知道`PID`，想用进程名作为参数，可以用这条：

```
taskkill /IM "xxx.exe" /F
```

如果你喜欢用`PowerShell`，还可以用下面两个命令：

```
Stop-Process -Name "xxx.exe" -Force
Stop-Process -ID 1024 -Force
```

分别对应了用名称或者`PID`杀进程。

---

后来我又遇到一种情况，即使用上面的方式还是不行。无意中发现可以这样：

![CleanShot 2022-04-11 at 17.28.53](/media/CleanShot%202022-04-11%20at%2017.28.53.png)

在任务管理器，选中杀不掉的进程，右键->`分析等待链`，

![CleanShot 2022-04-11 at 17.29.11](/media/CleanShot%202022-04-11%20at%2017.29.11.png)

然后可以把它列出的在`等待链`中的进程进行结束，就可以杀死之前不能杀死的进程了。



