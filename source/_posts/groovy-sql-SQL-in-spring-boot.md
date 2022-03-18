---
title: 在SpringBoot中使用groovy.sql.SQL高效开发
date: 2018-08-29 11:07:02
categories: 程序人生
tags:
    - Spring
    - groovy
---
某种情况下，其实就想用`Spring Boot`提供的那种即开即用的开发体验，但是我真的对`Spring`保姆式的一揽子工程不怎么感冒。尤其是数据库`JDBC`这块。常见的`Java`系里提到的技术，我真的一个都不想用，我最钟爱的数据库类库其实就是`groovy.sql.SQL`，简单而强大，配合`Groovy`之后，再也没有繁琐的`Java Bean`和无边无际的`get`、`set`。
想了解更多`groovy.sql.SQL`欢迎查看官方文档：http://groovy-lang.org/databases.html
今天我们还是着重说一下，怎么在`Spring Boot`的框架下融入`groovy.sql.SQL`，话不多说，上代码：
#### gradle.build
```gradle
plugins {
    id 'org.springframework.boot' version '1.5.15.RELEASE'
    id 'war'
    id 'groovy'
}

group 'com.aruistar'
version '1.0-SNAPSHOT'

ext {
    groovy_version = "2.5.2"
}

sourceCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
//    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")

    compile "org.codehaus.groovy:groovy:$groovy_version"
    compile "org.codehaus.groovy:groovy-sql:$groovy_version"
    compile "org.codehaus.groovy:groovy-json:$groovy_version"

    compile group: 'com.alibaba', name: 'druid', version: '1.1.10'
    compile group: 'mysql', name: 'mysql-connector-java', version: '5.1.46'

    testCompile "org.codehaus.groovy:groovy-test:$groovy_version"
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

#### 程序入口Application
```java
package com.aruistar

import com.alibaba.druid.pool.DruidDataSource
import groovy.sql.Sql
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder
import org.springframework.boot.builder.SpringApplicationBuilder
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.boot.web.support.SpringBootServletInitializer
import org.springframework.context.annotation.Bean

import javax.sql.DataSource

@SpringBootApplication
class Application extends SpringBootServletInitializer {

    static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }


    @Bean
    @ConfigurationProperties("app.datasource")
    DataSource dataSource() {
        return DataSourceBuilder.create().type(DruidDataSource.class).build();
    }

    @Bean
    Sql db() {
        return new Sql(dataSource())
    }
}

```

#### 配置文件application.yml
```yml
server:
  port: 8080

app:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test
    username: root
    password: 1
    pool-size: 5
```

#### 用上Sql的Controller
```java
package com.aruistar.controller


import groovy.sql.Sql
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("/open")
class OpenController {

    @Autowired
    Sql db

    @RequestMapping("/test")
    def test() {
        db.firstRow("select 1)
    }
      
}

```

#### 总结
这套方案用上了`Spring`，也用上了数据库连接池`druid`，所以项目整体是足够健壮的。如果你的项目本来就是基于`Spring`技术栈的，那我强烈推荐你试试结合`Groovy`的这套打法。可以大幅提升开发效率。
**亲自跑一下`Groovy`项目，用用她提供的`SQL`类，我想你会爱上她的**