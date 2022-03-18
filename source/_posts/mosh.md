---
title: ssh连接保活，mosh初体验
date: 2018-08-25 07:11:11
categories: 程序人生
tags:
    - Linux
---
传统的`ssh`最恼人的就是放着不动，没多久就丢了。有时候`tail -f`跟踪一个日志，一会日志没动静，可能`ssh`就已经阵亡了，而我还没反应过来，还琢磨着日志该出来了。烦。
前两天研究`iTerm2`的时候，看到有人分享`mosh`的相关知识，提到它最大的优点就是`ssh`保活。马上决定一试。
1. 服务端安装
```
yum install mosh
```

2. 防火墙放行端口，端口从`60000`排队起步，根据同时可能产生的连接数量，适当放行几个就好
```shell
firewall-cmd --add-port=60000-60010/udp --permanent
firewall-cmd --reload
```

3. 客户端同样安装`mosh`
```
brew install mosh
```

4. 客户端使用
    ```
    mosh -ssh="ssh -p 3322" root@192.168.0.88
    ```
    上面的`-ssh`命令主要是用在`ssh`端口不是开在`22`的情况下，需要指定端口。因为`mosh`自己的`-p`是指的`mosh`自身要用的`udp`端口。
    另外，如果你维护了`～/.ssh/config`文件的话，那么恭喜你，`mosh`可以直接读取这个`config`，也就是说，之前`ssh a_server`可以无缝替换成`mosh a_server`

5. 排错，如果遇到客户端访问不了服务器，并且有如下提示
    ```
    locale: Cannot set LC_CTYPE to default locale: No such file or directory
    locale: Cannot set LC_ALL to default locale: No such file or directory
    ...
    ...
    Connection to xxx.xxx.xxx.xxx closed.
    /usr/local/bin/mosh: Did not find mosh server startup message. (Have you installed mosh on your server?)
    ```
    可以尝试在服务器上的`~/.bashrc`文件追加这两行解决
    ```
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8
    ```

6. 最后说下实际体验感受
    的确大大延长了`ssh`的可用时间，但是像网上说的能坚持个个把月，很遗憾我并没有那么幸运，不过`mosh`仍然是个值得推荐的工具。