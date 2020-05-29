---
title: mouse-click-bubble.js,一个点击页面冒泡的小插件
date: 2018-04-28 19:01:05
tags: 
- js
- wheels
---
今天在逛博客，偶然发现人家页面上有点击冒个文字的效果。觉得挺好玩，给自己博客也加个吧  
一想原理，就是定位到鼠标所在位置，然后向上改变top，然后opacity:0，然后remove。
说干就干，jQuery10分钟完事。
这个效果挺好玩的，也挺好看的，为何不将他封装起来。正好巩固js面向对象这块的知识
## 思路分析
1. + this.value存储文字
  + this.span存储每一个创建出来的DOM对象
  + 让span动起来
  + 销毁span

2.看起来貌似没什么毛病（还是经验不太多呀）

## 遇到的问题
<!--more-->
1.prototype中创建的方法，内部调用不到，因为调用时，还未定义
	解决办法：
	+ 先创建，后执行
	+ ES6中class可轻松解决此问题（不得不说es6还是好用，还有Object.assign简直神器我的妈，太爽了）
2.setInterval中this指向问题。。。这个看到报错就解决了，不能说算问题吧。但是在写代码过程中没有预先处理这块，还是需要努力
3.耗时最长的一块：原生js动画setInterval的问题。
	总结setInterval写动画注意事项：
	+ 一进入函数，不管有没有，先clearInterval(timer)，不然会越点越快
	+ 判断停止条件，将timer写在else块中
	+ 像这种好多动画一起动的，绑定timer到obj自身，则不会互相影响
具体怎么动就不说了，position的top值
嗯，打开网页查看效果都没问题。nice！然而当点的快的时候，报错了！而且互相影响，越跑越快
WTF？明明已经obj.timer了啊，怎么会互相影响？

## 解决setInterval互相影响问题
1. 先观察效果，当一个点完消失之后，再点，完全没问题，就是在上一个还没消失的时候点，才会出现这个问题！
2. 查看控制台报错，removeChild报错
3. 检查代码，恍然大悟，卧槽，**this.span是共用的**
4. 如何能将每一个span挂到不同的this.span上？
5. 我试了将this.span设为数组，将每一个创建出来的span挂到不同的index下。想法听起来好像有一丢丢道理。有个屁，index怎么确定，还是公用的
6. 因为头一次写插件，这个问题我想了很久。。。
7. 终于！将和span有关的部分写到另一个类里，在MouseBubble类里每次创建span都去新实例化一个DoSpan不就好了吗！
8. 沃德天，完美！

再优化一下，加入自定义参数，类型判断，颜色、距离啥的可以自定义的东西

忙了一下午，总结出来这么寥寥几行字，还是有这么多坑需要踩，但是成就感爆强啊有木有。

写都写了，发到npm上吧

## 发布至npm
1. 老板，来个[npm账号](https://www.npmjs.com/signup)
2. 去文件夹下`npm init`
	+ `name` 你的包的名字(一定要先去npm官网看看有没有叫这个名字的包，有的话，重起名字)
	+ `version` 你的包的版本，今后每publish前都得更新版本
	+ `description` 你的包的描述
	+ `entry point` 你的包的主要js
	+ `test command` 不要管，回车
	+ `git respository` 如果你把你的包传到github上了，就把地址写上
	+ `keyword` 诶呦这个不得了了，这个跟网站的seo有的一拼(我是这么觉得的)，和你包有关的名词都写上吧
	+ `author` 你的名字
	+ `license` 回车
	+ `Is this OK?` (yes) 
3. 记得，你的主文件里必须要有module.exports要暴露出来，不然别人require谁呀
	一般都是，根目录放一个`index.js`，然后里面require你`lib`目录下的主js文件
4. 好，package.json里更新版本（第一次的话就不用了）
5. `npm publish`

## 项目引入
1. `npm install mouse-click-bubble --save`安装
2. 在vue项目中`const MouseClickBubble = require('mouse-click-bubble')`
3. new MouseClickBubble()

完美！
看下效果图
![](/images/11264410-7fcdbc801a43d351.gif)
其实就是博客现在的样子啦

放上地址吧，以后写的小轮子都放github了  [直达链接](https://github.com/pangjunpeng/myWheel)

