---
title: 用groovy实现根据规则校验单据数据
date: 2019-02-20 14:38:04
categories: 程序人生
tags:
    - groovy
---
单据字段校验，在任何管理信息系统中都是普通得不能再普通的问题。通常我们的都会选择在前端以及后端各实现一遍。
前端实现，方便用户体验；后端实现，确保数据健康。
但是总觉得同样的业务实现两遍，真的不符合直觉。我的想法是，把校验规则抽象出来，通过公共方法来处理单据校验问题。做到一套规则，两处判断。减少业务开发人员的重复劳动。
比如规则是这样的
```
[
    {
        "target": "v_name",
        "expression": "notBlank(v_name)",
        "tip": "姓名必填"
    },
    {
        "target": [
            "v_password",
            "v_password_twice"
        ],
        "expression": "v_password == v_password_twice",
        "tip": "两次密码输入不一致"
    }
]
```
这里面`target`是为了跟前端交互，在规则验证失败时给前端UI标红框使用的；`tip`是出错信息提示。只有`expression`才是核心业务——能返回`boolean`的表达式。
怎么才能驱动表达式执行呢，这在有动态特性的`groovy`中简直是小菜一碟。最简单的：
```
assert true == Eval.me("2>1")
```
轻松通过，但是注意我上面的`v_password == v_password_twice`这就比较棘手了，要从一个上下文中获取字段数据，再执行表达式，
这里假设我们准备的数据是：
```
def map = [
        "v_name"          : '',
        "v_password"      : "abc",
        "v_password_twice": "abc",
]
```
想让`groovy`动态脚本认识`map`，需要借助更强大的`GroovyShell`，代码如下：
```
def sharedData = new Binding()
def shell = new GroovyShell(sharedData)
map.each { k, v ->
    sharedData.setProperty(k, v)
}
assert true == shell.evaluate("v_password == v_password_twice")
```
看起来`Binding`可以轻易地构造执行脚本的上下文环境，那能不能更进一步，传入一个方法呢，比如上文的`notBlank`，也很简单
```
def notBlank = { String s ->
    s != null && s.trim().length() > 0
}
sharedData.setProperty("notBlank", notBlank)
assert true == shell.evaluate("notBlank(v_name)")
```
有了强大的`groovy`，就可以实现基于表达式的动态数据验证，没有占位符，没有字符串替换，还支持注入方法，真是即简单又强大。
这是我最终实现的方法：
```
/**
 * 检查数据是否符合规则
 * @param rules 规则List
 * @param map 待校验数据
 * @return 不满足规则的rules
 */
List check(List rules, def map) {
    def sharedData = new Binding()
    def shell = new GroovyShell(sharedData)
    sharedData.setProperty("length", { String s ->
        if (s == null) return 0
        return s.length()
    })
    shareData.setProperty("notBlank",{ String s ->
    s != null && s.trim().length() > 0
    })
    map.each { k, v ->
        sharedData.setProperty(k, v)
    }

    rules.findAll { !shell.evaluate(it.expression) }
}
```
完整测试代码参考：https://gist.github.com/aruis/d4a28b3cedfcc2a23a85ac67ca68adb7
