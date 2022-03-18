---
title: 基于Geb的京东抢券
date: 2018-08-05 08:33:13
categories: 程序人生
tags:
    - Geb
---
前面介绍过[Geb](http://gebish.org)是个好东西，可以用它实现web前端的自动化测试，自然用来解放双手，抢个券什么的不在话下。
这次我们主要瞄准京东。最佳的实验场所是京东的移动web版，即[https://m.jd.com/](https://m.jd.com/)，大概分为以下几个步骤：
1. 打开京东首页
2. 点击登录按钮
3. 填写登录用户名密码并登录
4. 调整到需要抢券的页面
5. 找到抢券按钮开抢

下面贴一个，618某活动的抢券代码（现在已经下线了），仅供参考。
```
 static def checkin(String username, String password) {


        FirefoxBinary firefoxBinary = new FirefoxBinary();
//        firefoxBinary.addCommandLineOptions("--headless");
        FirefoxOptions firefoxOptions = new FirefoxOptions();
        firefoxOptions.setBinary(firefoxBinary);

        def browser = new Browser(driver: new FirefoxDriver(firefoxOptions))

        browser.with {

            go "https://m.jd.com"

            $(".jd-search-icon-login").click()

            $("#username").value(username)
            $("#password").value(password)

            $("#loginBtn").click()

            sleep(5000)

            go "https://pro.m.jd.com/mall/active/qKRVTAJL7v93L71TkJebPv5GJnE/index.html"


            $("#m_1_14").children().each {
                it.click()
                sleep(500)
            }


        }
    }
```
补充几个说明：
1. 我用的是`Firefox`浏览器。所以需要电脑事先装好该浏览器。
2. 只装浏览器还不够，还需要相应的`WebDriver`驱动。如果找不到程序会报错的，可以根据错误提示下载该驱动。然后要把本地驱动文件地址设置一下，方便`Geb`识别，代码如下` System.setProperty("webdriver.gecko.driver", "/root/geckodriver")`
3. **无节目的Linux服务器可不可用呢？答案是可以**，在安装完上面两样东西之后，只需要代码里添加`firefoxBinary.addCommandLineOptions("--headless");`就可以启动无界面的`Firefox`了

#### 友情提醒
工具抢券也好、代码抢券也好，都是一种对普通消费者不公平的存在。建议学会之后，自用练手即可，切勿以此牟利。