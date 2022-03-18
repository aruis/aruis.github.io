---
title: 免开发环境在GitHub站点上快速给开源项目贡献代码
date: 2018-08-18 14:13:27
categories: 程序人生
tags:
    - GitHub
---
给开源项目贡献代码应该算是进阶程序员的一条必经之路。今天我就简单介绍一下，如何在不clone代码，不使用本地开发环境的情况下，给GitHub上的开源项目贡献代码。
还没有GitHub账号的小伙伴，就抓紧注册个吧，其他就没有任何必要条件了。

### 第一步，发现问题
这个我可教不了，只能介绍一下我自己的经验，就是多看，多想。比如我今天举例的就是`JVM`下知名项目`Vert.x`，官方维护的[vertx-examples](https://github.com/vert-x3/vertx-examples)，我发现他在介绍`gradle-redeploy`的时候，其中`build.gradle`有一段写得就不够严谨，他是这么写的：

```
// Vert.x will call this task on changes
def doOnChange
if (System.getProperty("os.name").toLowerCase().contains("windows")) {
  doOnChange = '.\\gradlew classes'
} else {
  doOnChange = './gradlew classes'
}
```
看过我之前博客的小伙伴应该知道，操作系统的文件分割符，`Java`中是提供静态方法等多种方式来获取的，不需要通过自己手动判断操作系统的名字来自己实现。所以我决定把这段代码优化一下。也就是这篇博客的起因。

### 第二步，`Fork`代码
点这里即可：
![](/media/%E7%B2%98%E8%B4%B4%E7%9A%84%E5%9B%BE%E7%89%872018_8_18_14_23.png)
`Fork`成功之后，会自动进入到自己命名空间下的项目
![](/media/15345735455740.jpg)

### 第三步，创建分支
`GitHub`上的`Pull Request`都是基于分支的，也就是说想要贡献代码，要先创建一个分支，在上面承载你的代码变更。其实做起来也相当简单，如图：![](/media/%E7%B2%98%E8%B4%B4%E7%9A%84%E5%9B%BE%E7%89%872018_8_18_14_28.png)

### 第四步，修改代码
之后就会进入到自己新建的分支，找到要修改的代码后，点编辑按钮，如图![](/media/%E7%B2%98%E8%B4%B4%E7%9A%84%E5%9B%BE%E7%89%872018_8_18_14_32.png)

把我之前说的代码改成：

```
def doOnChange = ".${File.separator}gradlew classes"
```
一行搞定，是不是简单多了。
编辑完代码后，在屏幕最下方，提交代码，注意要写一个理由充分的提交注释，证明你改变代码的必要性
![](/media/15345741245811.jpg)

### 第五步，发起`Pull Request`
把修改`commit`之后，回到自己的项目首页，就可以看到一句提示，问你要不要把刚修改的那个分支去跟原始项目比较，并且发起`pull request`，如图![](/media/%E7%B2%98%E8%B4%B4%E7%9A%84%E5%9B%BE%E7%89%872018_8_18_14_39.png)

点击这个按钮后，进入最后的`Open a pull request`界面，再次要把提交说明写到位，就可以点`Create pull request`按钮正式发起贡献代码请求了。

### 其他
1. 基本开源项目都有自动化的代码审查，比如单元测试、格式规范检查之类的，发起`PR`之后也要留意观看，万一发现自己代码有问题，还可以继续更改
2. 之后还回到自己分支，产生的更改，就不用重新发起`PR`了，只要`commit`之后，之前的`PR`可以直接看到
3. 一旦`PR`成功，就可以在原始项目的`contributors`中找到自己的账户了。希望你到时候可以榜上有名；）