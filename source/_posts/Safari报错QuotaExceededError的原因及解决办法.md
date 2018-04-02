---
title: Safari报错QuotaExceededError的原因及解决办法
date: 2018-03-30 08:47:02
tags: bug
comments: true
---
## 首先，这个报错是在IOS的safari浏览器的无痕模式下才有的！！

昨晚项目上线，本来以为一切正常10点多就可以走了，没想到，没想到啊没想到，上去之后经理把我叫过去，“你看一下”。。。**沃德天**这！怎！么！可！能！  

于是引入了个VConsole(真是神器)查看报错，`QuotaExceededError`，这是个什么鬼，从来没见过，一番查找之后，发现原来是safari开了隐私模式（无痕）限制了`Storage`的锅（经理你为什么要开隐私模式！）狗日的测试测了这么久都没测出来  

>  Safari在所谓的`private mode`下不允许使用`LocalStorage`功能，只有在用户自身开启`non-private mode`的情况下才可以正常使用`LocalStorage`。

## 原因
Safari的无痕模式下，`Storge`对象依然存在，但是就是不能`setItem`，不能`removeItem`(那要你有何用)

## 解决办法
知道原因解决起来就简单多了
+ 判断`Storage`对象是否可用
+ 不可用，使用`cookie`替代