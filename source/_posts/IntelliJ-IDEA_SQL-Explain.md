---
title: IntelliJ IDEA中的SQL Explain
date: 2018-07-31 08:11:45
categories: 程序人生
tags:
    - IntelliJ IDEA
---
善于利用`SQL`的`explain`是`SQL`调整优化的必经之路。但是遇到复杂的`SQL`，查看`explain`结果也是有点困难的。比如这样的：
![](/media/15329969004233.jpg)
好在很多数据库的客户端都提供了图形化的表现形式，比如`PostgreSQL`的`pgAdmin4`就能看到如下效果：
![](/media/15329972711011.jpg)
不过如果有个工具，能够hold住所有主流的关系型数据库的话，就更嗨皮了。答案就是`IntelliJ IDEA`。如果你还没有尝试过其自带的`Database`功能的话，推荐现在就试一下。在屏幕右侧应该能找到。
![](/media/15329976430336.jpg)
只需要按照向导添加相应的数据库连接就好了。不过这个功能免费的社区版是没有的。
创建完数据库链接后，通过此按钮打开`SQL Console`窗口
![](/media/15329978378217.jpg)
然后在里面就可以愉快的编写`SQL`了。
先来一段：
![](/media/15329983218004.jpg)
现在我们可以尝试通过`IntelliJ IDEA`执行一次`explain`了。
* 先把光标移动到需要`explain`的`SQL`上
* 右键呼出菜单，找到![](/media/15329984643618.jpg)   执行之
* 就能看到效果了![](/media/15329985590089.jpg)
* 点击`Show Visualisation`可以呼出图形化展示![](/media/15329987110873.jpg)
* 如果觉得这个功能实用的话，可以给它设置一个快捷键，方法如下
    * 呼出`Find Action...`窗口（快捷键：⇧⌘A），在`Help`菜单下能找到
    * 搜索到`explain`![](/media/15329989448215.jpg)
    * 按快捷键⌥↩︎，或者windows下的`alt+回车`
    * 就可以设置一个快捷键了![](/media/15329990816786.jpg)









