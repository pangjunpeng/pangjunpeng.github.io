---
title: hexo主题yilia中tag标签内特殊字符被转义的解决办法
date: 2018-04-09 15:39:24
tags: 
- Q&A
- bug
- hexo
---
今天在发布文章时添加了个新标签`Q&A`,然后就发现
![](https://user-images.githubusercontent.com/27885402/38476291-775a8f4e-3bdf-11e8-8f0d-78875aa2cdf5.png)
`&`被转义为`&amp;`了，所以这条tag就索引不出来  
<!--more-->
一时间不知道问题所在，就去看litten大神的[《Yilia源码目录结构及构建须知》](https://github.com/litten/hexo-theme-yilia/wiki/Yilia%E6%BA%90%E7%A0%81%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%E5%8F%8A%E6%9E%84%E5%BB%BA%E9%A1%BB%E7%9F%A5)，可以发现控制这部分的代码应该在`source-src`目录中，控制台F12可以发现input中model值为search  

![](/images/11264410-a8c44a171db55fda.PNG)  

于是  

![](/images/11264410-a2ecd150e31fdb86.PNG)  

```app.$set('search', '#' + (name ? name : e.target.innerHTMl))```  

就是他了！
控制台测试果然`innerHTML`会进行转义，为什么呢！  

查阅红宝书，发现
> &emsp;&emsp;为 innerHTML 设置 HTML 字符串后，浏览器会将这个字符串解析为相应的 DOM树。因此设置了 innerHTML之后，再从中读取HTML字符串，会得到与设置时不一样的结果。原因在于返回的字符串是根据原始 HTML 字符串创建的 DOM 树经过序列化之后的结果。

这就很明显了，`innerHTML`会先解析为DOM树，而`&amp;`正是html中的转义符，改为`innerText`便可以正常操作了，不过这样就不能自定义一些标签样式了（不过也没这样用过），如果想要innerHTML的效果，可以使用replace方法哈哈。  

**但是**，改完之后好像并没有生效，上面那行代码根本没有调用啊喂，真正的凶手在下面
![](/images/11264410-e6123d62a44242be.PNG)
把下面这个`$em.innerHTML`改了就可以了  

**记得改完之后npm run dist编译一下，当然你也可以直接去dist文件中改（如果你找到了的话）**