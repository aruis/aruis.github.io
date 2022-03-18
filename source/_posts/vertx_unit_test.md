---
title: 给异步的Vert.x程序做单元测试
date: 2018-09-13 08:33:56
categories: 程序人生
tags:
    - Vert.x
---
异步的程序先天不好单元测试，尤其是按照传统的`JUnit`思路来弄，肯定是不行的。好在`Vert.x`想到了这一点，所以提供了`vertx-unit`包，专门考虑了对异步代码的测试。
使用起来也很简单，首先
```
testCompile "io.vertx:vertx-unit:3.5.3"
```
然后创建一个测试类
```
@RunWith(VertxUnitRunner.class)
public class JUnitTestSuite {
  @Test
  public void testSomething(TestContext context) {
    context.assertFalse(false);
  }
}
```
如果要是测试异步程序，只需要调用`TestContext`的`async()`方法即可。然后直到手动调用其`complete()`方法，整个测试过程才会结束。请看例子：
```
@Test
void testSomething(TestContext context) {
    def async = context.async()
    Vertx.vertx().setTimer(3000, {
        context.assertFalse(false)
        async.complete()
    })
}
```
`async`方法的注释写得还是很详细的
```
Create and returns a new async object, the returned async controls the completion of the test. Calling the Async.complete() completes the async operation.

The test case will complete when all the async objects have their Async.complete() method called at least once.

This method shall be used for creating asynchronous exit points for the executed test.
```

更多内容请参考官方文档：[vertx-unit](https://vertx.io/docs/vertx-unit/java/)