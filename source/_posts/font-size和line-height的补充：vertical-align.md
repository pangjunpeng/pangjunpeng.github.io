---
title: font-size和line-height的补充：vertical-align
date: 2018-08-01 11:10:35
tags: css
---
上一篇[经典布局之姓名-联系方式对齐](http://pangjunpeng.com/2018/07/24/%E7%BB%8F%E5%85%B8%E5%B8%83%E5%B1%80%E4%B9%8B%E5%A7%93%E5%90%8D-%E8%81%94%E7%B3%BB%E6%96%B9%E5%BC%8F%E5%AF%B9%E9%BD%90/)中解释了`font-size`和`line-height`以及字体设计的关系，今天看到张鑫旭大神的[CSS深入理解vertical-align和line-height的基友关系](https://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)才发现，上回理解的只是冰山一角，文章挺长，看完做个总结怕忘喽
<!--more-->

上回书说道，设计师在设计字体的时候会设置字体默认行高以及空格的宽度，所以导致不同浏览器空格宽度不同，以及div高度飘忽不定的问题

------------------------分割线------------------------
我们都遇到过一个div中只放一个img会出现一条缝的问题，解决办法有很多
1. 页面上去掉空格使标签紧挨住（排版不好看）
2. div设置font-size为0（子元素如需要得重新设置）
3. vertical-align: top（其他值也可）
4. 接下来说
可见前两个都有相应的缺陷（当然也不是说vertical没有缺陷，只是为了引入今天的话题），vertical-align顺带还解决了内联元素垂直居中的问题（虽然很别扭）

### 以前我一直以为，那条缝是标签之间的空隙（空格），今天我发现我错了，缝隙是由看不见的`幽灵元素`的行高撑起来的，而不是说那条缝本身就是空格

![](/images/11264410-b3c00c0d8193aed0.png)
这张图清晰明了的说明了这个问题
	`vertical-align`默认是`baseline`基线对齐，也就是`x`的下边缘
再回顾字体默认行高的知识，就能更清晰地知道问题所在，所以解决缝隙又多了一条解决办法，而上面的`font-size: 0`也只是变相的改变了`line-height`而已
```
5. div{ line-height: 5px; } //小于字体大小就可以
```
再者，缝隙是由于`vertical-align`和`line-height`联手造成的，也可以从`vertical-align`入手，而`vertical-align`对block元素无感，所以设置图片为block也可解决此问题
```
6. img{ display: block; }
```

### 衍生功能：垂直居中
设置line-height，让幽灵元素vertical-align: middle;就可以了(吗?)，据张老师说，绝对居中的位置和字体的middle不是一个位置，字体会略微下沉，也就是说我们看起来的垂直居中其实不居中，那怎么办呢，还是老办法`font-size`，设为0，管他什么基线上线下线，全在一条线上，不就居中了吗，哈哈，张老师真是牛逼

### 还有个进阶知识
	一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。

头疼，这句话已经说得很明白了，总的来说，任何莫名其妙出现的对不齐问题都可以用font-size: 0解决，因为它可以让各种线归于一处，可以让line-height变为0，当然，后遗症什么的还得慢慢填坑，所以有时候并不是一个好的解决办法

想深入了解透彻的话，感觉就得涉及字体设计方面了

先这样吧，趁月初需求没分下来赶紧学习写两篇博客记录记录，月中有个临时投产，呀好烦哦
![](/images/11264410-663e948ef75d831b.jpg)
