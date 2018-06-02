---
title: html中不识别四个方块符号？自己画一个！
date: 2018-05-18 18:44:34
tags: js
---
![](https://upload-images.jianshu.io/upload_images/11264410-682635b5c3e6432b.png)
图中标题前的 长得像梅花 的四个方块  
业务给了个文档让做成页面（为什么不做成微信帖子），文档里有这么个标志，选取不到，但是复制-粘贴到编辑器的竟然好使，还心想挺智能的嘛，结果浏览器上直接为，emmm......
### 心路历程：
<!--more-->
+ 因为文档有好几个小标题，遵循语义化的规则，就选用ul>li，然后就想找找list-style有没有这个形状的，很遗憾，找了一圈，没有！
+ list-style-image？看起来不错，截个小图放上
![](https://upload-images.jianshu.io/upload_images/11264410-aa2a6a8ab8c4eae1.png)
看起来不错哦，但是！放到手机上有点小模糊，身为一个前端工程师，肩负着用户体验这一神圣使命，怎么能容许模糊？？？
+ 算了自己画一个吧，看起来也不是很难。  
一个span包裹四个i，变为inline-block定宽，i给margin背景色，然后transform:rotate(45deg)，bingo~
代码就不放了，就是这个效果，看起来也很不错，但是一想，这个页面里面我可以给class重复使用，如果以后也有项目要用呢，我再写一遍？虽然代码量很少...谈不上一点复用性健壮性，如果一行字特别小，定的这个高度会不显得特别不协调，岂不是还要针对这一个单独修改？能不能做成自适应行高的，说来就来

```javascript
/* 获取高度
 * 因为高度没有在标签中style属性内定义，所以直接点取不到
 * 先写个获取属性的方法
 * 再来获取高度
 */
 
 // 获取计算后的样式
 function getStyle(el){
   return window.getComputedStyle?window.getComputedStyle(el, null):el.currentStyle
 }
 // 获取父元素高度，设span宽高为父元素高度的一半。html结构是这样的<h1><span></span> 这是一个标题</h1>
 function setSide(el){
   var pHeight = getStyle(el).height
   el.style.width = el.style.height = pHeight.slice(0, height.length - 2) / 2 + 'px'
 }
```
看起来好像好了，但是传el还是有点。。。难看啊,而且如果有多个呢，还得自己写for
不如只传一个id或class或tag就好了
再来
```javascript
function getEl(str){
  if(str.startsWith('#')){
	return document.querySelector(str)
  }else if(str.startsWith('.')){
	return document.querySelectorAll(str)
  }else {
	return document.querySelectorAll(str)
  }
}
```
将`getEl`加入`setSide`，判断是HTMLElement还是NodeList来决定是否遍历
```javascript
var el = getEl(el)
if(!el.length){
  //生成梅花
}else{
  for(var i = 0; i < el.length; i++){
	//生成梅花
  }
}
```
有重用代码，重新定义一个函数叫`createListStyle`，重用setSide部分，于是
```javascript
function createListStyle(){
    var el = getEl(el)
	if(!el.length){
	  setSide(el)
	}else{
	  for(var i = 0; i < el.length; i++){
		setSide(el[i])
	  }
	}
}
```
这样，重用问题就解决了，但是如果要用冲的话，总不能让人去写四个i标签吧，还有样式肯定得我们自己定，不然还这重用有啥意义，来吧
```javascript
//写入setSide中，这样的话这个名字就不行了，改叫setListStyle吧
el.style.cssText = 'display:inline-block;margin-right:10px;transform: rotate(45deg);font-size: 0;'
var pHeight = getStyle(el.parentNode).height
el.style.width = el.style.height = pHheight.slice(0, height.length - 2) / 2 + 'px'

el.innerHTML = '<i></i><i></i><i></i><i></i>'
el.childNodes.forEach(function(item){
  item.style.cssText = 'display: inline-block;width:34%;height:34%;margin:8%;background:#000;'
})
```
到目前为止，就没有什么问题了，可以直接调用createListStyle('.your-className')
### 但是！
万万没想到的是，当这行标题超过一行，变成两行或多行时！这个玩意会变得很大，跟其他的显得特别不协调
想办法：
+ emmm...检测他是否超过一行？查询无果，怎么可能有这种方法嘛（阿摔
+ 能否设置他的高为父元素的line-height值？这个想法不错，但是一获取，结果竟然是‘normal’，wtf？？？
+ 再想不到其他办法了T_T，或者说代码结构还可以再设计的合理一点？用rem？可是并不知道其他人的1rem是多大
算了，定宽吧，可是又不能一味全定，毕竟作为一个标题换行的情况还是很少的。  
给createListStyle添加第二个参数side传入边长，内部判断是否传此参数，传了就定宽
```javascript
if(side){
  el.style.width = el.style.height = side
}else{
  el.style.width = el.style.height = pHeight.slice(0, pHeight.length - 2) / 2 + 'px'
}
```
至此，便可以通过单独设置id的方式来单独设置他的高度了

不过到这里，代码显然是比直接写多了好几倍。不过万一那个项目中用的多了呢，嘻嘻

显然还可以写成面向对象式的，代码如下
```javascript
    function GridListStyle(params){
      Object.assign(this, params)
      this.createListStyle(this.el, this.side)
    }
    GridListStyle.prototype.createListStyle = function(el, side){
      var el = this.getEl(el)
	  //检验是否有此元素，拦截后续报错
      if(!el || Object.prototype.toString.call(el).match('NodeList') && el.length === 0){
        console.log('未发现此元素')
        return false
      }
      if (!el.length) {
        this.makeListStyle(el, side)
      } else {
        for (var i = 0; i < el.length; i++) {
          this.makeListStyle(el[i], side)
        }
      }
    }
    GridListStyle.prototype.makeListStyle = function (el, side) {
      el.style.cssText = 'display:inline-block;margin-right:10px;transform: rotate(45deg);font-size: 0;'
      var pHeight = this.getStyle(el.parentNode).height
      if (side) {
        el.style.width = el.style.height = side
      } else {
        el.style.width = el.style.height = pHeight.slice(0, pHeight.length - 2) / 2 + 'px'
      }
      el.innerHTML = '<i></i><i></i><i></i><i></i>'
      el.childNodes.forEach(function (item) {
        item.style.cssText = 'display: inline-block;width:34%;height:34%;margin:8%;background:#000;'
      })
    }
    GridListStyle.prototype.getEl = function(el){
      if (el.startsWith('#')) {
        return document.getElementById(el.slice(1))
      } else if (el.startsWith('.')) {
        return document.getElementsByClassName(el.slice(1))
      } else {
        return document.getElementsByTagName(el)
      }
    }
    GridListStyle.prototype.getStyle = function(ele){
      return window.getComputedStyle ? window.getComputedStyle(ele, null) : ele.currentStyle
    }
    new GridListStyle({
      el: '.list-style',
      side: '12px'
    })
```
到此为止，这个功能应该就挺完善了。  
其实我觉得对于新手来说，以后能不能用得上并不是最主要的，关键是能不能从一个小的功能上，把他抽离出来可复用。  
有了这种思想，以后遇到大工程中重用性强的组件，必定会顺手的多。勿以用小而不为，路得一步一步走。加油吧~