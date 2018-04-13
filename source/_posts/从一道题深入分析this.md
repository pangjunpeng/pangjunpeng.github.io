---
title: 从从三个题渐渐深入this
date: 2018-04-12 14:53:58
tags: js
---
> **this**
> 在函数执行时，this 总是指向调用该函数的对象。要判断 this 的指向，其实就是判断 this 所在的函数属于谁。
> 在《javaScript语言精粹》这本书中，把 this 出现的场景分为四类，简单的说就是：
> 有对象就指向调用对象
> 没调用对象就指向全局对象
> 用new构造就指向新对象
> 通过 apply 或 call 或 bind 来改变 this 的所指。

上面这段话相信大部分同学初学js时都听过，那时候感觉this好麻烦，还要分四种情况讨论，每次看到this的题都一脸懵逼，总是做错。  
屡屡网上寻找this文章，得到的却总是千篇一律的套话，他复制她，她复制他

**最近闲暇之时研究了一下this**，发现一句话
<!--more-->
> + fn()只不过是fn.call()的缩写
> + obj.fn()不过是obj.fn.call(obj)的缩写

其实这句话本意和`谁调用指向谁`没有本质区别，但是转换成代码，却通俗易懂的多。

`fn.call()`因为没有指向任何对象，所以指向了`undefined`，而**非严格模式**下，指向`undefined`和`null`都会被js引擎转换为**`window`**。所以如果
```javascript
'use strict'
function fn(){
    console.log(this)
}
fn()//undefined
```
在严格模式`'use strict'`下，this则指向了**undefined**

**相信把上面这段话看懂后，已经不怕大部分的this题了**
接题：
```javascript
var myObject = {  
    foo:"bar",  
    func: function() {  
        var self = this;  
        console.log("outer func: this.foo = " + this.foo);  
        console.log("outer func: self.foo = " + self.foo);  
          
        (function (){  
            console.log("inner func: this.foo = " + this.foo);  
            console.log("inner func: self.foo = " + self.foo);  
        }());  
    }  
}  
myObject.func();  
```
+ 首先前两个肯定是一样的，同一作用域，只是赋了个值而已。
+ 此题中self指向myObject，因为`myObject.func.call(myObject)`，，所以前两个应该是`bar`
+ 而下面这个匿名函数，匿名函数的this是谁呢，谁调用了？**前面没有点！**，所以指向**window**，所以this指向window
+ 那么self呢？
    + 有同学可能说this都是window了，self当然是window.self啦，所以应该是undefined！&emsp;&emsp;&emsp;**××××××××**
    + 只能说。。**瞎猫碰上死耗子**
    + **闭包！**（敲黑板！！！！）
    + self不同于this，**this是作为函数的参数存在的**，而self是一个实实在在的变量，而变量的值，是不看this指向的。匿名函数虽然指向this，但不代表他就在window作用域下执行！他只是undefined被转为window了而已。
    + 所以，此时，self应该按照词法作用域此次往上找（从哪儿开始找？从定义函数的地方开始！）匿名函数往上一层找到了name值为func的函数，发现了`self`，然后使用，**所以`self`指向为myObject**
所以答案应该为
```
//outer func: this.foo = bar
//outer func: self.foo = bar
//inner func: this.foo = undefined
//inner func: self.foo = bar
```
不要停，看下面这个
```javascript
var number=2;
var obj={
  number:4,
  fn1:(function(){
    var number;
    this.number*=2;
    number=number*2;
    number=3;
    
    return function(){
      var num=this.number;
      this.number*=2;
      console.log(num);
      number*=3;
      alert(number);
    }
  })(),

  db2:function(){  
    this.number*=2;  
  }  
}  
  
var fn1=obj.fn1;  //(1)
  
alert(number);  //(2)
  
fn1();  //(3)
obj.fn1();  //(4)
alert(window.number);  //(5)
alert(obj.number);  //(6)
```
控制台调试后是不是发现又错了（滑稽.png）  

为方便理解，对上面主要执行语句标了标记，来看看  

(1)：`var fn1 = obj.fn1;`  
将`obj.fn1`的值赋给`fn1`，`obj.fn1`是什么呢，是个[自执行函数](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)，上面我们已经知道其this指向window，所以这步`window.number === 4`,同时声明局部变量`number`并赋值为3，然后返回一个function(){}，function里面是啥，我们先不看，因为还没有执行。  
**语句（1）执行完之后得到：**
+ `window.number === 4`
+ 局部变量`number===3`

(2)：`alert(number);`  
显而易见，弹出window.number  
    ```
        alert(4) //输出结果(1/7)
    ```

(3)：`fn1();`  
fn1是什么，是在（1）处声明的变量，所以他的值为（1）后来返回的那个function(){}，也就是
    ```javascript
    fn1 = function(){
      var num=this.number;
      this.number*=2;
      console.log(num);
      number*=3;
      alert(number);
    }
    ```
**this是啥？**
`fn1.call()`，当然是window，所以局部变量`num`值为window.number的值。
**那`number`呢？**
当前作用域内没有number呀，当然是去定义函数的地方层层往上找（不懂看上题，**闭包啊！！**），而在步骤（1）时，number值已经为3了，所以此时找到的number也为3，所以`number*=3;`之后，number值变为9！所以
**语句（3）执行完后得到**
+ `num === 4`
+ `window.number === 8`
+ 局部变量`number === 9`
```
    console.log(4) //输出结果(2/7)
    alert(9) //输出结果(3/7)
```

(4)：`obj.fn1();`  
看到obj.fn1()，二话不说，`obj.fn1.call(obj)`，this指向obj，所以this.number为4(obj.number)，局部变量number的值呢？当然是9了，因为声明是在父函数进行的，子函数调用赋值，父函数当然会被改变，当下一次再找到时，父函数已经变了，所以`number*=3`后，局部变量number变为27。
**语句（4）执行之后得到**
+ `num === 4`
+ `obj.number === 8`
+ 局部变量`number === 27`
```
    console.log(4) //输出结果(4/7)
    alert(27) //输出结果(5/7)
```

(5)：`alert(window.number);`
window.number从步骤（3）后没有过更改，所以
    ```
        alert(8) //输出结果(6/7)
    ```

(6)：`alert(obj.number);`
obj.number在步骤（4）时更改，所以
    ```
        alert(8) //输出结果(7/7)
    ```


**不知道我有没有讲清楚，如果有同学看了之后还有不明白的地方，欢迎评论或左下角骚扰**