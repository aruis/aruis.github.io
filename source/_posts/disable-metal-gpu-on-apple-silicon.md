---
title: 在Apple Silicon上关闭M1的GPU，仅用CPU进行Tensorflow训练
date: 2022-04-18 20:13:48
categories: 程序人生
tags:
    - DeepLearning
---

在苹果`M1`系列芯片上运行`tensorflow`是可以通过插件`tensorflow-metal`进行`GPU`训练加速的，并且随着操作系统的升级以及插件的不断完善，`M1`的训练性能正在稳步提高，这也是苹果官方推荐的做法。

不过某些情况下，我们还是需要关闭`GPU`加速，仅使用`CPU`进行训练。那么以下这段代码可以帮助你临时禁用`GPU`，而把训练的压力转移到`CPU`上来。

```
import tensorflow as tf

tf.config.set_visible_devices([], 'GPU')
```