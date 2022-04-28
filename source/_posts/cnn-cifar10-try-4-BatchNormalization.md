---
title: 通过BatchNormalization把CIFAR10的训练精度提升到85%
date: 2022-04-21 16:56:36
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

在不借助数据增强的情况下，我们已经一路把验证精度从`70%`多提升到了`80%`的水平，今天，我们借助`BatchNormalization`可以进一步把精度提升到超过`85%`。话不多说，先看结果：

![CleanShot 2022-04-21 at 16.59.36](/media/CleanShot%202022-04-21%20at%2016.59.36.png)

![CleanShot 2022-04-21 at 17.00.10](/media/CleanShot%202022-04-21%20at%2017.00.10.png)

通过观察每轮的数据可知，在最后的10轮的训练中，都保证了验证精度超过`85%`，已经算是不错的成绩了。下面是模型的代码：

```python
from keras import layers
from keras import models


model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.2))

model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.3))

model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.BatchNormalization())
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.4))

model.add(layers.Flatten())
model.add(layers.Dropout(0.4))
model.add(layers.BatchNormalization())
model.add(layers.Dense(10, activation='softmax'))

model.summary()
```

对比上次的模型，唯一的区别就是增加了若干的`BatchNormalization`层。那咱们就来聊聊它。

`BatchNormalization`，中文叫做`批量归一化`，实际上是一种正则化，其功能就是对每个`mini-batch`正则化，使数据分布的均值为0、方差为1，以此调整数据的广度。该方案于2015年被提出，并深受大家的喜爱。据书中所述，它有如下几个有点：

* 可以增大学习率，也就是学得更快。
* 不那么依赖初始值。
* 抑制过拟合。