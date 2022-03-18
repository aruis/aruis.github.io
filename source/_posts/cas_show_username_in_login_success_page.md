---
title: 如何在cas登录成功页面显示用户名
date: 2018-07-20 08:14:24
categories: 程序人生
tags:
    - cas
---
[CAS](https://www.apereo.org/projects/cas)这种本来业务场景就很细分，再加上其上古时代存续至今的特质。估计还在用它的公司已经不多了。间接导致，其中文资料比较匮乏。
领导说，想再cas成功后的欢迎页，也就是`casGenericSuccess.jsp`页面，可以显示`欢迎:xxx`的字样。
不得不说，这个需求非常常规。然而不仅cas默认没有实现，甚至翻遍google，都很难找到满意的答案。比较有参考价值的可能就是[google groups](https://groups.google.com/forum/#!topic/jasig-cas-user/q_pjYXCe7ko)上的这篇。居然要借助额外的jar包(还是个已停止维护的)才能实现。不得已，只能自己想办法。
现在给出我的思路：
1. 在`deployerConfigContext.xml`文件中，找到`authenticationHandlers`参数，其应该对应一个类。十有八九，那个类是你自己实现的，如果不是，可以自己继承一下原有参数配置的类。
2. 然后在那个类里的`authenticate`方法，可以通过追加`HttpSession session = RequestContextHolder.getRequestAttributes().getSessionMutex().session as HttpSession`这么一行，获取到`session`，这就嗨皮了。
3. 可以在`authenticate`方法需要返回`true`的时候，之前增加一行`session.setAttribute("username",balabala);`，这样我们就顺利把username塞到`session`里的
4. 最后，修改`casGenericSuccess.jsp`，在需要显示用户名的地方，加入`<%=session.getAttribute("username")%>`，就可以实现在登录成功页面显示用户名了
5. 如果想显示更复杂数据内容，可以留意下`deployerConfigContext.xml`里面的`credentialsToPrincipalResolvers`所对应的类。它的时间节点是判定用户登录成功之后，组织用户信息用的。
