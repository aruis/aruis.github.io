---
title: 使用简单的CNN训练CIFAR10，理解padding='same'的含义
date: 2022-04-14 16:01:53
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

今天在训练`CIFAR10`数据的时候，稍微调整了下网络，对卷积层增加了`padding`，最终结果得到了一定的改善：

![CleanShot 2022-04-14 at 20.05.02](/media/CleanShot%202022-04-14%20at%2020.05.02.png)

上图左边，是增加了`padding='same'`的结果，右图是[上一次](https://www.kankanzhijian.com/2022/04/12/cnn-cifar10/)的模型。可以看到左边的训练精度明显好于右侧。而验证精度也略好于右侧。

参考`keras`的文档：

> **padding**: one of "valid" or "same" (case-insensitive). "valid" means no padding. "same" results in padding with zeros evenly to the left/right or up/down of the input. When padding="same" and strides=1, the output has the same size as the input.

`padding`设置为`true`后，会对输入的上下左右进行填充，并且如果`strides`（步幅）设置为`1`后，输入与输出的大小应该相同。如果拿`4*4`的输入为例，用`3*3`的卷积以步幅为`1`进行运算时。应该在上下左右四边每边补充一组数据，形成一个`6*6`的输入，这样在和`3*3`做以步幅为`1`做卷积的时候，输出才会是`4*4`（6-3+1=4）。

回到本例，`CIFAR10`的原始大小是`32*32`的，如果没有`padding='same'`，那么在与`3*3`做步幅为`1`的卷积时，输出的大小就应该是`32-2`，也就是`30*30`了。

关于`padding`参数的解释就到这，至于为什么增加`padding`之后，训练的结果朝着更好的方向发展。或者说，如果不提供`padding`，会使训练结果变差，在`《深度学习入门 基于Python的理论与实现》`这本书中有这样的解释：

> 如果每次进行卷积运算都会缩小空间，那么在某个时刻输出大小就有可能为1，导致无法再应用卷积运算。

基于这种极限化的考虑，那么在每次卷积之后，保证输出大小不变就是必要的了。

不过我在测试API使用中还遇到一个问题，如果`strides`大于`1`，那么`padding='same'`也是无法保证输出与输入大小相同的。这个虽然在`keras`文档里说了，但是我却有点困惑，觉得这有悖于`same`的字面意思，只能留着以后再研究了。

最后附上今天使用的核心代码：

```python
from keras import layers
from keras import models


model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.Flatten())
model.add(layers.Dropout(0.5))
model.add(layers.Dense(512, activation='relu'))
model.add(layers.Dropout(0.5))
model.add(layers.Dense(10, activation='softmax'))

model.summary()

model.compile(optimizer='Adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])


history = model.fit(train_images, train_labels, epochs=50, batch_size=64,validation_data=(test_images, test_labels))
```
