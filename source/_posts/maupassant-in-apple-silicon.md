---
title: maupassant主题在Apple M1上不能用的问题
date: 2022-03-22 17:12:44
categories: 程序人生
tags:
    - Mac
---

我这个博客是用[hexo](https://hexo.io/zh-cn/) 搭建的，说实话`node`的技术栈真是很烦，各种依赖非常重，借用一位朋友的话说"天下苦`node`久矣"。

不过对我来说，平时开发也不用`node`了，现在就用用`blog`嘛，也不是不能忍。

谁成想，两年前换了`Apple M1`后，情况都变了。`blog`直接跑不起来了，当时我一直以为是`hexo`的问题，很无奈就只能停更了。

中间陆陆续续，各种东西也在适配，我也都顺手尝试了几次，不过无疾而终，这也就导致了我这个`blog`断更了这么久。

`失之东隅，收之桑榆`，虽然`blog`都长草了，但是[B站](https://space.bilibili.com/24370353) 倒是有点蒸蒸日上的感觉。

时间来到了`2022`年，我突然觉得视频的形式虽然受众广，也适合安利教程啊、软件啊、开箱评测类的东西。但是本质上，视频对沉淀自己，其实没有`blog`来的深刻。

所以`blog`还是不能丢，从今天起，我要回归这个`blog`，继续记录生活与技术。

回归标题，说说我是怎么解决之前我的`blog`无法`hexo g -d`的问题。

经研究发现不是`hexo`的问题，而是我用的这款皮肤`maupassant`，它依赖一个包`hexo-renderer-sass`，众所周知，`sass`的一些生态在`node`里就是万恶之源啊。

但其实`sass`大部分情况下也早就适配`M1`了，只是这个`hexo-renderer-sass`已经三年没人管了。实在没办法，我只能自己`fork`了一下，改了一下依赖。

也就是把下面这段：

```shell
"dependencies": {
 "node-sass": "^4.5.3"
}
```

改成了:

```shell
"dependencies": {
 "node-sass": "^7.0.1"
}
```

这个时候，回到我自己的`blog`项目的`package.json`文件，手动添加依赖：

```shell
"dependencies": {
  ...
  "hexo-renderer-sass": "github:aruis/hexo-renderer-sass",
  ...
}  
```

然后`npm install`，就万事大吉了。

就这样，我的`blog`就复活了，😄
