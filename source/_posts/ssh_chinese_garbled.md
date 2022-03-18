---
title: ssh中文乱码
date: 2018-09-02 10:50:27
categories: 程序人生
tags:
    - 乱码
---
ssh远程到服务器后，遇到中文乱码，建议通过`locale`检查当前的字符集，如果遇到`LC_ALL=`无值，十有八九是要出问题的。
此时可以通过执行`export  LC_ALL=zh_CN.UTF-8`临时解决。
当然，把上面这句添加到`.bash_profile`中，就可以永久解决了。 就像这样
```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
export  LC_ALL=zh_CN.UTF-8
```

补充一点，用`en_US.UTF-8`代替上文的`zh_CN.UTF-8`也是同样有效的。