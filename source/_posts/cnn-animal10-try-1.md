---
title: 简单训练一下Animal-10的数据（第一版，精度很拉垮）
date: 2022-04-29 09:50:23
categories: 程序人生
tags:
    - Keras
    - DeepLearning
---

之前一直用[CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html)的数据做训练。这个数据已经内置到`Keras`里了，所以拉取数据很方便，只需要一行`cifar10.load_data()`即可。但是在实际工作中，最终肯定还是用自己的图片作为素材进行训练，所以这次决定下载[Animals-10](https://www.kaggle.com/datasets/alessiocorrado99/animals10)作为训练素材。

![CleanShot-2022-04-29-10.01.19](/media/CleanShot-2022-04-29-10.01.19.png)

可以看到这个数据集的图片质量普遍不错，分辨率在300*300左右，但是尺寸比例比较原始，参差不齐，正适合来做学习研究。

首先需要做的对原始数据进行切分，分成`训练集`、`验证集`和`测试集`。代码如下：

```python
import os, shutil

# 原始文件路径
original_dataset_dir = '/Users/liurui/develop/deeplearning/animal10'

# 转移到的新路径
base_dir = '/Users/liurui/Downloads/animal10'
# shutil.rmtree(base_dir)
os.mkdir(base_dir)

train_dir = os.path.join(base_dir, 'train')
os.mkdir(train_dir)
validation_dir = os.path.join(base_dir, 'validation')
os.mkdir(validation_dir)
test_dir = os.path.join(base_dir, 'test')
os.mkdir(test_dir)

list_file = os.listdir(original_dataset_dir)

min_size = 10000
for i in range(0,len(list_file)):
    animal_name = list_file[i]
    path = os.path.join(original_dataset_dir,animal_name)
    img_file = os.listdir(path)
    size = len(img_file)

    if size < min_size :
        min_size = size

print(min_size)

for i in range(0,len(list_file)):

    animal_name = list_file[i]
    # 构造路径
    path = os.path.join(original_dataset_dir,animal_name)

    train_path =  os.path.join(train_dir,animal_name)
    validation_path =  os.path.join(validation_dir,animal_name)
    test_path =  os.path.join(test_dir,animal_name)

    os.mkdir(train_path)
    os.mkdir(validation_path)
    os.mkdir(test_path)

    img_file = os.listdir(path)
    # size = len(img_file)
    size = min_size

    for i in range(0,size):     
        fname = img_file[i] 
        if fname.endswith(".png") :
            continue

        src = os.path.join(path, fname)
        dst = os.path.join(train_path, fname)    

        if i > round(size/2):
            dst = os.path.join(validation_path, fname)

        if i > round(size/4 * 3):                        
            dst = os.path.join(test_path, fname)
            
        shutil.copyfile(src, dst)

```

因为是第一次接触这个数据集，考虑到每个`类别`的图片数量不一致，又为了保证每个类别的数据相对平衡，所以我以最小的`类别`数量为基准准备数据。

同时还有个尴尬的问题就是数据集中包含一些`png`图片，不多，也就几十张，但是因为这些图片的存在导致训练过程中报警告，所以我决定把它们都给剔出去。

文件夹准备好后，我们就可以准备`keras`里面的`训练集`等对象了。

```python
from tensorflow.keras.preprocessing import image_dataset_from_directory

train_ds = image_dataset_from_directory(
    directory=train_dir,
    labels='inferred',
    label_mode='categorical',
    batch_size=32,
    image_size=(150, 150))
    
validation_ds = image_dataset_from_directory(
    directory=validation_dir,
    labels='inferred',
    label_mode='categorical',
    batch_size=32,
    image_size=(150, 150))
```

以`train_ds`为例，它其实就是我们程序里要跑的`训练集`。
* `directory`指向的是我们准备好的文件夹，已经按类别分好类的，像这样：

![CleanShot-2022-04-29-10.13.43](/media/CleanShot-2022-04-29-10.13.43.png)

* `labels='inferred'`就是让它把上图的文件夹当作标签。
* `label_mode='categorical'`就是说我们要做多个分类（大于2），如果是`二分类`问题可以设置为`binary`。
* `batch_size=32`，是说这个生成器按批生成，每批的大小，`32`本身也是这个参数的默认值。
* `image_size`作为图片的大小就好理解了，毕竟我们要构建的网络`input`是固定的，所以这些元素图片必须整合成统一的大小。

下面开始构建网络：

```python
from keras import layers
from keras import models


model = models.Sequential()
model.add(layers.Rescaling(1./255))
model.add(layers.Conv2D(32, (3, 3), activation='relu',padding='same'))
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
```

整个网络跟我们之前用的改动不大，主要集中在一开始。首先加入了`model.add(layers.Rescaling(1./255))`，用来把图片的`RGB`一个通道的值压缩到小于等于1的`浮点`型，这也是所有网络建议的输入数据。然后就是删除了之前我们模型里写的`input_shape=(32, 32, 3)`代码，因为这次咱们的大小也不是`32*32`了，而且`keras`本身可以自动推断这个`input`，所以还是删掉为妙。

剩下的就是训练了，跟之前也区别不大：

```python
model.compile(optimizer='Adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])


history = model.fit(
      train_ds,      
      epochs=30,
      validation_data=validation_ds
      )
```

直接看结果：

![CleanShot-2022-04-29-10.28.45](/media/CleanShot-2022-04-29-10.28.45.png)

三个字**不太行**，`5`轮之后就过拟合了，一直到接近`30`轮，验证精度还达不到`55%`。我们会在下次调整网络继续进行测试。

