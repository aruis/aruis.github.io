---
title: Vert.x项目监测代码变化自动冷重启
date: 2018-08-14 09:15:17
categories: 程序人生
tags:
    - Vert.x
---
传统的Java Web开发一般都会提供热重载的方式，方便开发人员在代码发生变化的时候，无需手动重启应用，就可以刷新到效果。用惯了的人，在用`Vert.x`开发的时候多少会有点不习惯，不过`Vert.x`程序启动速度还是很可观的，所以也勉强能忍。
后来通读文档的时候，发现有个关于`redeploy`的介绍，似乎能用，又似乎不好用的，直到看到官方的[Vert.x 3.2 Gradle redeploy project](https://github.com/vert-x3/vertx-examples/tree/master/gradle-redeploy)总算豁然开朗了。
核心代码无非下面几行
```
// Vert.x watches for file changes in all subdirectories
// of src/ but only for files with .java extension
def watchForChange = 'src/**/*.java'

// Vert.x will call this task on changes
def doOnChange
if (System.getProperty("os.name").toLowerCase().contains("windows")) {
  doOnChange = '.\\gradlew classes'
} else {
  doOnChange = './gradlew classes'
}

run {
  args = ['run', mainVerticleName, "--redeploy=$watchForChange", "--launcher-class=$mainClassName", "--on-redeploy=$doOnChange"]
}
```
我也尝试了一下，的确可用，但是这个官方示例还是有个地方写的不好，就是关于文件分割符的判断，恰好这个在我之前的帖子里面提到过，见[编写跨操作系统Java代码时动态获取文件分隔符](https://www.kankanzhijian.com/2018/07/26separator_in_java/)
所以上面的`gradle`代码关于`doOnChange`声明就可以简化成下面的样子了
```
def doOnChange = ".${File.separator}gradlew classes"
```
测试成功

