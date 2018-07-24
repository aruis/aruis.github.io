---
title: 用JMH做简单的JVM性能测试
date: 2018-07-14 06:39:47
categories: 程序人生
tags: Java
---
写java也有年头了，有时候遇到对比某几种方法性能的情景，都是自己傻傻的打印`new Date()`计算时间差。现在想来，这种原始的方式，就跟不会用IDE Debug，只会`System.out.println()`打印调试没什么区别。
这次被人安利`JMH`，说来惭愧，是我在Twitter上质疑`Vert.x`采用了效率不高的Json序列化库，影响了其在[techempower](https://www.techempower.com/benchmarks/)的成绩。结果[@julienviet](https://twitter.com/julienviet)神回复我说"you should make a JMH microbenchmark to find out"，所以才有了这篇，利用JMH做Json序列化速度对比的文章。
这是测试结果：
![-w783](/media/15315228604634.jpg)
可以看到`Vert.x`的Json序列化速度还是出类拔萃的，当然，其本质还是实用的`jackson`
上代码：
```java
package com.aruistar.benchmark;

import com.aruistar.benchmark.model.User;
import com.jsoniter.output.JsonStream;
import groovy.json.JsonBuilder;
import groovy.json.JsonOutput;
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;


public class JsonToStringBenchmark {

    public static void main(String[] args) throws RunnerException {

        Options opt = new OptionsBuilder()
                .include(JsonToStringBenchmark.class.getSimpleName())
                .forks(1)
                .warmupIterations(2)
                .measurementIterations(3)
                .build();

        new Runner(opt).run();
    }


    //    @Benchmark
    public void testJsonObjectToBuffer() {
        new User("Hello, World!", "tomcat", 10, "angular", true).toBuffer();
    }

    @Benchmark
    public void testJsonObjectToString() {
        new User("Hello, World!", "tomcat", 10, "angular", true).toString();
    }


    @Benchmark
    public void testJsonBuilder() {
        new JsonBuilder(new User("Hello, World!", "tomcat", 10, "angular", true).getMap()).toString();
    }

    @Benchmark
    public void testJsonOutput() {
        JsonOutput.toJson(new User("Hello, World!", "tomcat", 10, "angular", true).getMap());
    }

    @Benchmark
    public void testJsoniter() {
        JsonStream.serialize(new User("Hello, World!", "tomcat", 10, "angular", true).getMap());
    }


}

```

```java
package com.aruistar.benchmark.model;


import io.vertx.core.json.JsonObject;

public class User extends JsonObject {


    public User(String name, String username, int age, String title, boolean bool) {
        put("name", name);
        put("age", age);
        put("title", title);
        put("bool", bool);
    }


}

```

源码地址，[https://github.com/aruis/somebenchmark](https://github.com/aruis/somebenchmark)

需要注意的是，如果在IDEA打开，想直接通过main方法启动，需要执行如下步骤
```
Do you have org.openjdk.jmh:jmh-generator-annprocess on your classpath?
If yes, is annotation processing enabled in your IDE? You can find the checkbox under
Preferences -> Build, Execution, Deployment -> Compiler -> Annotation Processors
```
* 参考[https://github.com/artyushov/idea-jmh-plugin/issues/13]