---
title: 用Gradle自动发布程序至Linux服务器
date: 2018-07-18 09:41:26
categories: 程序人生
tags:
    - gradle
---
你的重复劳动，一定能找“人”帮你做，聪明的程序员一定是拒绝重复的。由于公司条件限制，暂时用不上`jenkins`，先拿`gradle`救救急也是不错的。今天要实现的是，通过gradle发布静态站点到服务器。这样可以和上回的[用Gradle打包Vue前端程序](http://www.kankanzhijian.com/2018/07/17/用Gradle打包Vue前端程序/)保持一定的连贯性。当然本帖拿来发布`war`包也是ok的。
* 首先追加gradle插件，参考写法

```
plugins {
    id 'org.hidetake.ssh' version '2.9.0'
}
```
或者
```
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'org.hidetake:gradle-ssh-plugin:2.9.0'
  }
}
apply plugin: 'org.hidetake.ssh'
```
* 配置插件

```
remotes {
    webServer {
        host = '192.168.1.100'
        user = 'develop'
        port = 22
        password = 'xyz'
    }
}
```
如果觉得密码服务器密码明文写在这里不安全，可以用公钥ssh的方案，那么这里可以用`identity = file('id_rsa')`代替掉`password = 'xyz'`

* 配置完Gradle SSH Plugin，就可以自己写`task`实现上传文件了。下面贴段我的

```
task deployPortal {
    group = 'release'
    dependsOn zipPortal
    doLast {
        ssh.run {
            session(remotes.webServer) {
                put from: "$buildDir/portal.zip", into: "/home/develop/"
                def result = execute 'unzip -o  /home/develop/portal.zip -d /home/develop/portal/'
                println(result)
            }
        }
    }
}
```
这个task的主要作用就是把压缩好的静态站点上传至服务器，然后再解压缩。重点就两句话，第一句`put from: "本地文件", into: "服务器路径"`，实现文件上传功能。第二句`execute '执行shell命令'`，实现通过shell命令，解压缩文件。
* 至于中间那句`dependsOn zipPortal`表示执行发布task之前，先要把文件准备好，这个`zipPortal`task我是这么写的
    ```
    task zipPortal(type: Zip) {
        dependsOn(':portal:build')
        from 'portal/www'
        archiveName 'portal.zip'
        destinationDir buildDir
    }
    ```
*  换句话说，如果你是要上传`war`包的话，可能就不是`dependsOn zipPortal`而是`dependsOn war`了。其他的地方，大同小异。