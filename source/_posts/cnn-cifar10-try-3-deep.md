---
title: 通过加深网络把CIFAR10的训练精度提升到80%
date: 2022-04-15 13:51:06
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

这次继续，在原来网络的基础上，加深了卷积层的数量，从原来的3层卷积，加深到了6层。核心代码如下：

```python
model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))

model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))

model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Flatten())
model.add(layers.Dropout(0.5))

model.add(layers.Dense(512, activation='relu'))
model.add(layers.Dropout(0.5))
model.add(layers.Dense(10, activation='softmax'))
```

![CleanShot 2022-04-15 at 13.52.32](/media/CleanShot%202022-04-15%20at%2013.52.32.png)

可以看到验证精度比之前的略好一些。但是程度非常有限，刚刚有接近`80%`的影子。下面我尝试增加`Dropout`层：

```python
model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.2))

model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.3))

model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Flatten())
model.add(layers.Dropout(0.4))


model.add(layers.Dense(512, activation='relu'))
model.add(layers.Dropout(0.5))
model.add(layers.Dense(10, activation='softmax'))
```
在原来的基础上增加了两层`Dropout`，结果点进一步得到改善：

![CleanShot 2022-04-15 at 13.57.20](/media/CleanShot%202022-04-15%20at%2013.57.20.png)

已经达到`80%`的水平了。

---

到这之后，我又发现一个自己代码的问题，之前修改代码的时候，去掉了最后卷积层之后的池化层，肯定是不妥的。我现在给加回来：

```python
from keras import layers
from keras import models


model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.2))

model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.3))

model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.4))

model.add(layers.Flatten())
model.add(layers.Dense(512, activation='relu'))
model.add(layers.Dropout(0.5))
model.add(layers.Dense(10, activation='softmax'))
```

重新训练结果如下：

![CleanShot 2022-04-15 at 14.22.21](/media/CleanShot%202022-04-15%20at%2014.22.21.png)

可以看到验证精度最后都能超过`80%`。只能说怪我了，卷积层后面不跟池化层的话数据就太大了，用池化层压缩一下，训练效果往往会更好。

下面我又尝试把`Flatten`之后的全连接层给去掉，得到了更好的数据：

![CleanShot 2022-04-15 at 14.40.38](/media/CleanShot%202022-04-15%20at%2014.40.38.png)

训练在`12`轮之后，验证精度就稳步在`80%`以上了。
最终代码如下：

```python
from keras import layers
from keras import models


model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3),padding='same'))
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.2))

model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(64, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.3))

model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.Conv2D(128, (3, 3), activation='relu',padding='same'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Dropout(0.4))

model.add(layers.Flatten())

model.add(layers.Dense(10, activation='softmax'))

model.summary()

model.compile(optimizer='Adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])


history = model.fit(train_images, train_labels, epochs=50, batch_size=64,validation_data=(test_images, test_labels))
```