---
title: 使用VGG16训练CIFAR-10的一次失败记录
date: 2022-04-01 17:23:56
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

最近研究`深度学习`，主要想做计算机视觉相关的事情。因为我是个纯新手，所以想把一些学习进展记录一下。毕竟`深度学习`的超参数设置，还是非常依赖经验的，而我在这方面只是个小白，如果我能把每次训练的情况，都详细记录下来，或许能够帮助我总结一些经验。毕竟跑一次训练，少则几分钟，多则数十分钟（我是M1 Air，没风扇，也不敢跑太久的），既然这时间都花了，所以结果还是记录的越详细越好。

`CIFAR-10`是一组别人准备好的图片数据集，总共有10个类别，每个类别有`6000`张图像，但是每个图片都不大，才`32*32`。

今天我先用别人已经预训练好的模型`VGG16`做个基准，后面再找机会看看能不能改进吧。先来看结果：

![CleanShot 2022-04-01 at 17.42.06](/media/CleanShot%202022-04-01%20at%2017.42.06.png)

这明显是`过拟合`了，模型在`训练集`上表现越来越好，但是在`验证集`上精度就是上不去。我估计还是因为`VGG16`的模型比较大，参数多，所以在训练集不充足的情况下比较容易过拟合。下面是我训练相关的代码：

```python
import keras
from keras.datasets import cifar10
from tensorflow.keras.utils import to_categorical
from tensorflow.keras.applications import VGG16

(train_images, train_labels), (test_images, test_labels) = cifar10.load_data()

train_images = train_images.astype('float32') / 255
test_images = test_images.astype('float32') / 255
train_labels = to_categorical(train_labels)
test_labels = to_categorical(test_labels)

conv_base = VGG16(weights='imagenet',
                  include_top=False,
                  input_shape=(32, 32, 3))
                  
conv_base.trainable = True

set_trainable = False
for layer in conv_base.layers:
    if layer.name == 'block5_conv1':
        set_trainable = True
    if set_trainable:
        layer.trainable = True
    else:
        layer.trainable = False
        
from keras import layers
from keras import models


model = models.Sequential()
model.add(conv_base)
model.add(layers.Flatten())
model.add(layers.Dense(512, activation='relu'))
model.add(layers.Dense(10, activation='softmax'))


model.compile(optimizer='rmsprop',
              loss='categorical_crossentropy',
              metrics=['accuracy'])


history = model.fit(train_images, train_labels, epochs=25, batch_size=64,validation_data=(test_images, test_labels))
```

下面是绘制精度图的代码：

```
import matplotlib.pyplot as plt

acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(len(acc))

plt.plot(epochs, acc, 'bo', label='Training acc')
plt.plot(epochs, val_acc, 'b', label='Validation acc')
plt.title('Training and validation accuracy')
plt.legend()

plt.figure()

plt.plot(epochs, loss, 'bo', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.legend()

plt.show()
```

今天就这样，回头我会尝试改进模型训练，看能不能把精度提高一点。