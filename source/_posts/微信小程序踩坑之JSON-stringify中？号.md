---
title: 微信小程序踩坑之JSON.stringify中？号
date: 2018-07-30 14:23:18
tags: 
- js
- bug
---

首先吐槽一下微信小程序的跳转页面，竟然没有传值，传值只能通过拼接url取参数，wtf，虽然还可以用storage，但总觉得传值这个东西用storage就有点。。大材小用？也不对。人家本意就不是来干这个的，非得让人家干，虽然也能干。  
本着不信邪的态度，找了半天资料，得，gg，老老实实拼接字符串吧，但是我要传的是个对象啊没办法，JSON.stringify吧，问题也随之而来

navigator到另一个页面后再JSON.parse竟然报错，说这不是一个json，打印出来看了一下，我的数据竟然被吃了一半，什么鬼，那么多shit一样的api都度过来了，竟然在这小阴沟里帆船，这怎么解。

其实仔细一看，我穿的数据中有一条是带着参数的url，按理说他JSON.stringify不应该出这问题啊，回头一想，他传参的方式使用url参数传递的，恍然大明白！

**问题并不在JSON.stringify这，转的很顺利。但是当另一个页面接受的时候可不是全盘接受**  

1. 他取的参数。
2. 参数中有`?`
由此可推断出，onload函数中参数取值应该是`location.href.split('?')[1]`，完全没有考虑到参数中会有?的情况，相信在不久的将来这个api会更新的，用location.search取值然后再split('&')多好，当然也可能是有原因才这么做的，同为程序员，理解理解。

推断出问题原因所在，开始解决问题，分割?，那我们就把我们的？转义掉，用到了api`encodeURIComponent`，传之前，先把有url的参数转义了，在接受的时候再`decodeURIComponent`转义回来，顺利解决。

微信的坑真的是。。堪比上古时代IE

![](https://upload-images.jianshu.io/upload_images/11264410-987eaa118aa4752e.jpg)
