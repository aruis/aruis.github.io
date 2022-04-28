---
title: 通过Data Augmentation把CIFAR10的训练精度提升到89%
date: 2022-04-28 10:42:36
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

我们先来回顾一下上一次的训练结果：

![CleanShot-2022-04-28-10.42.58](/media/CleanShot-2022-04-28-10.42.58.png)

可以看到在训练不到`20`批的时候，训练精度就与测试精度分道扬镳了。这算是一种过拟合。目前我们手上的工具箱也就剩`数据增强`还没用了。理论上`数据增强`可以弥补`训练集`太小的问题，从而缓解`过拟合`的现象。实话说，在实际操作中，这种方法已经被检验过有效了。但是总给人一种`用一种机器去欺骗另一种机器`的感觉，我个人觉得，`机械化`的`数据增强`应该早晚被更优秀的训练模型所取代。

闲话少说，我们来看下最终的效果：

![CleanShot-2022-04-28-10.52.00](/media/CleanShot-2022-04-28-10.52.00.png)

这是训练了`100`批的效果（足足花费了我43分49秒），前几次的训练都是到`50`批我就觉得差不多了。这次 一开始我也是用的`50`，但是当我发现`训练精度`被`验证精度`咬得死死的时候，我就又训练了`100`批的，也就是上面的结果。

可以看到，虽然从`40`批开始，训练进步的幅度就开始放缓了，但是整个训练还一直向好的方向上前进，`过拟合`现象也得到了抑制。基本在`100`批的时候，我们已经有了一个`验证精度`在`88%`～`89%`的模型，总的来说还是有效果的。

下面看下`数据增强`的核心代码：

```python
from keras.preprocessing.image import ImageDataGenerator

datagen = ImageDataGenerator(
        featurewise_center=False,  # 将整个数据集的均值设为0
        samplewise_center=False,  # 将每个样本的均值设为0
        featurewise_std_normalization=False,  # 将输入除以整个数据集的标准差
        samplewise_std_normalization=False,  # 将输入除以其标准差
        zca_whitening=False,  # 运用 ZCA 白化
        zca_epsilon=1e-06,  # ZCA 白化的 epsilon值
        rotation_range=0,  # 随机旋转图像范围 (角度, 0 to 180)
        # 随机水平移动图像 (总宽度的百分比)
        width_shift_range=0.1,
        # 随机垂直移动图像 (总高度的百分比)
        height_shift_range=0.1,
        shear_range=0.,  # 设置随机裁剪范围
        zoom_range=0.,  # 设置随机放大范围
        channel_shift_range=0.,  # 设置随机通道切换的范围
        # 设置填充输入边界之外的点的模式
        fill_mode='nearest',
        cval=0.,  # 在 fill_mode = "constant" 时使用的值
        horizontal_flip=True,  # 随机水平翻转图像
        vertical_flip=False,  # 随机垂直翻转图像
        # 设置缩放因子 (在其他转换之前使用)
        rescale=None,
        # 设置将应用于每一个输入的函数
        preprocessing_function=None,
        # 图像数据格式，"channels_first" 或 "channels_last" 之一
        data_format=None,
        # 保留用于验证的图像比例（严格在0和1之间）
        validation_split=0.0)

datagen.fit(train_images)
```

上面的代码基本上就是照抄`keras`官方文档的[一节](https://keras.io/zh/examples/cifar10_cnn/)，不过我们的模型与文档还是有很大不同，毕竟是我们一步步探索出来的，最终模型代码如下：

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

有了`数据增强`后，我们训练模型的代码要稍加改动：

```python
model.compile(optimizer='Adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])


history = model.fit(datagen.flow( train_images, train_labels,batch_size=64), epochs=100,validation_data=(test_images, test_labels))
```

最后再说一下，我用的`optimizer`（优化器）是`Adam`，跟上面提到的`keras`[文档](https://keras.io/zh/examples/cifar10_cnn/)里用的`RMSprop`还是有区别的。根据我实际测试下来，`Adam`的学习速度还是要好于`RMSprop`的。

![CleanShot-2022-04-28-13.25.26](/media/CleanShot-2022-04-28-13.25.26.png)

左图是`RMSprop`，右图是`Adam`（也就是我们上面实际在跑的结果）。通过对比我们可以看出来，`10`批之后，`Adam`的`训练精度`就达到了`75%`，而`RMSprop`的`训练`精度还仅有`64%`。所以把`Adam`作为一个默认选项会是一个挺好的选择，这也是我们在前几次训练中使用的。
