---
title: Groovy迭代器中的return陷阱
date: 2018-09-10 15:04:53
categories: 程序人生
tags:
    - groovy
---
`groovy`从不会让人失望，如果有，那就是接下来我要说的这种情况：
```
def list = [1, 2, 3, 4, 5, 6, 7]
def test(List list) {
    list.each {
        if (it > 3) {
            return it
        }
    }
}

println test(list)
```
我们期望的结果`4`，也就是找到第一个比3大的数字就返回了，然而事与愿违，得到的结果是这样
```
[1, 2, 3, 4, 5, 6, 7]
```
整个list都被打印了出来，其实`test`方法最终返回的东西是`list.each{...}`，而且`.each`方法的返回值是`the self List`，所以最后把`list`又原封不动打印了一遍也就不足为奇了。
那我们写的那行`return`干吗了，答案是，它既不会是方法返回，也不会使`each closure`返回，它在这里的作用跟`continue`类似，仅仅是让它后面的代码不在这次循环执行，仅此而已。

#### 那怎么才能打断一个`each`，然后让上面的方法提前`return`呢？
**答案是，不能用`each`，得用`find`或者`any`之类的，**有明显截断语义的迭代器。这里还是首推`find`，因为`any`返回值是`boolean`，能帮我们截断迭代，但是不适用于找东西的场景。

用`find`改造一下代码，如下
```
def test(List list) {
    return list.find {
        if (it > 3) {
            return it
        }
    }
}
```
由于`groovy`方法中的`return`是可以省略的，更进一步，就写成这样
```
def test(List list) {
   list.find {
        if (it > 3) {
            return it
        }
    }
}
```
当然基于`find`的先天特性，那句`return it`也可以省略成这样
```
def test(List list) {
   list.find { it > 3 }
}
```
再次之前执行`println test(list)`，得到结果`4`，一切归于完美了。

### 题外话
这里不得不说下`Kotlin`，它应付上面描述的场景就非常得心应手了。首先，默认情况下`return`就是代表让一个方法返回，这符合一个程序员的直觉。
在不想扩大`return`的打击范围，而仅仅只想结束一个`lambda`的时候，可以使用`标签返回`，类似这样
```
list.forEach{
    if (it > 3) {
        return@forEach
    }
}
```
如果上面的语法糖，让你觉得有点故弄玄虚了，还可以使用更容易理解的匿名函数，就像这样
```
list.forEach(fun(it){
    if (it > 3) {
        return
    }
})
```

**我是非常喜欢`groovy`的，她已经陪伴我8个年头了，几乎我手上所有的项目都离不开`groovy`的身影。我甚至用她写过`Android`程序。不过不得不承认，`Kotlin`作为后起之秀，的确在很多方向上更进一步，考虑的场景也更为全面。期待`groovy`的`3.0`能够发力一波。**