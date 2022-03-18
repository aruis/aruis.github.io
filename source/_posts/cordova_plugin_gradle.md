---
title: Cordova插件中定制build.gradle的方法
date: 2018-08-06 09:23:31
categories: 程序人生
tags:
    - Cordova
---
编写`cordova`插件的时候，有时候要有进一步设置`build.gradle`文件的需求，比如追加个依赖什么的。这中问题，可以通过设置`cordova`的`plugin.xml`来解决的。分为如下几个步骤：
1. 编写cordova.build文件，文件名可以随便叫，内容就放你需要追加的个性化内容，比如我的是
    ```
    android {
        sourceSets {
            main {
                jniLibs.srcDirs = ['libs']
            }
        }
    }
    ```
2. 在`plugin.xml`文件中`<platform name="android"></platform>`区域内追加配置
    ```
    <framework src="src/android/cordova.gradle" custom="true" type="gradleReference"/>
    ```
    
更多信息可以查阅文档 https://cordova.apache.org/docs/en/latest/plugin_ref/spec.html