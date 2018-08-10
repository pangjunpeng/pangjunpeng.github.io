---
title: 总结一下js中那些看起来没问题却不合规的语法
date: 2018-08-10 12:05:37
tags: js
---
今早在地铁上看《你不知道的javascript》，看到42.toFixed()，不是之前不知道，只是看到了，就将所有这种类似的问题总结一下
<!--more-->

**一、42.toFixed()**  
js会将`.`之后看做42.的小数部分，所以会解析不通过，报`Invalid or unexpected token`错，跟`42toFixed`错一样
**有效语法：**
```
(42).toFixed()  简单粗暴  

42..toFixed()  这是什么鬼！不要忘了`.9 === 0.9`,`1.0 === 1.`，js中头部尾部0是可以省略的，所以就会把42.先解析为42，已经解析过了，自然可以调用toFixed了  

42 .toFixed()  加个空格，使`42`和`.`不衔接，42为一个整体，自然不会报错啦  

另：小数不会有此问题：`0.6.toFixed()` 因为已经有个`.`了
```
**二、{}.hasOwnProperty('xxx')**  
js中`{}`是一个独立的代码块，这段代码相当于`.hasOwnProperty() `，什么鬼，所以会报语法错误`Uncaught SyntaxError: Unexpected token .`  
**有效语法：**  
不让引擎将`{}`当成代码块，用`()`包住：`({}.hasOwnProperty('xxx'))`

**三、解构赋值**  
```javascript
let x
{x} = {x: 1}
// Uncaught SyntaxError: Unexpected token =
```
看起来没问题呀。js中，`{}`如果放在句首，会被解析为单独的代码块，所以最后效果会跟`= {x: 1}`一样，这是什么鬼，所以当然会报语法错误啦  
**有效语法：**  
```javascript
let x
({x} = {x: 1})
```
其实这个问题还碰到过好几次，当我们真正理解之后，写代码时就可以及早避免这些问题

**四、说完了括号，是不是看起来括号真牛逼，简直万能，但是括号()也有坑！如下：**  
```javascript
var a
var y = 1+a
(a+y).toString()
// Uncaught TypeError: a is not a function
```
怎么回事，他把第二行的a和下面一行的()连起来了，但是
```javascript
function test(){
	return 
	1
}
```
这个却不会连起来，`return undefined`。。看样子只有括号才会连起来
```javascript
x
++
y
```
这他娘又是啥，`x + +y`，就变成了`x+y`了，所以啊，到底什么时候他不会自己换行呢，emmm......
1. 明显不能作为一行的
2. 下一行有括号的  

目前就这两点。  
所以我们在写IIFE的时候，一定要记住前面加一个能让引擎识别为独立表达式的标识，比如：  
`!(function(){})()`取反
`+(function(){})()`转number
`-(function(){})()`转number
`~(function(){})()`按位取反
等等   

说了这么多，总之，还是加上分号好，可以避免很多不必要的麻烦  

但是我还是不想加。。。