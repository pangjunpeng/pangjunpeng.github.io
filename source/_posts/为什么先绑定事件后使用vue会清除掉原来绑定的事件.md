---
title: 为什么先绑定事件后使用vue会清除掉原来绑定的事件
date: 2018-12-17 16:59:17
tags: 
  - js
  - vue
---
本来这篇文章应该是一个月前写的，奈何太忙，每天12点回，隔三差五的通个宵。最近好容易有空，赶紧补一补，记下来。

事情是这样的，项目是jsp，里面只有jquery。写一些简单的东西是很爽，维护起来想骂娘。

刚来大约一周多，正赶上一个版本提测，就跟着改bug，其中有个功能没有做。然后就我来做，也没多难。  
是这样的，密码框要做一个可见控件，就这种。
![](https://upload-images.jianshu.io/upload_images/11264410-4b72acf5772aad42.png?imageMogr2/auto-orient/)
![](https://upload-images.jianshu.io/upload_images/11264410-9e39e1480d66127a.png?imageMogr2/auto-orient/)
倒也没多难，用jq写就太操蛋了。而且在改了一周jq的bug后，我快疯了。  
其实jq倒也没啥，主要是原来的架构，同一个东西，完全没有复用起来，改一个东西得全局搜索，要改3处，甚至5处。你说恶不恶心！！！

本来一开始想的先熟悉项目熟悉业务，加vue的事情后面再说。但是这个功能，让我决定，立刻，现在，马上，刻不容缓。于是我尝试在jsp中引入vue.js，试了一个demo发现是可行的

于是在我加入下一版本开发的时候，就提枪上阵，引入vue，梳理项目结构。当然其中避免不了出现很多问题。印象最深的就是加载顺序的问题。
<!--more-->
过程是这样的，新建`v-home.js`（因为项目本没有babel，想用es6，就加了一个babel，而其他文件杂乱不堪，又不能全部babel，所以加入前缀方便babel判断解析），加入页面底部
```javascript
<script type="text/javascript" src="static/js/home/v-home.js"></script>
```
然后
```
new Vue({
  el: '#main',
  // ........
})
```
一切正常！非常开心

但是页面有个公共头部`header.jsp`这个上面是有跟多操作的，这个就加不上vue了，只好单独给`header.jsp`加上`v-header.js`，嗯！但是报错了。
```
Vue is not defined
```
很明显，`header.jsp`在文档上边，先解析，`v-header.js`加载的时候，还没有下面的vue，所以报了`Vue is not defined`，怎么办呢，又想在上面写，又想让他后加载。那就在DOMContentLoaded里执行呗，所以改造如下
```
document.addEventListener('DOMContentLoaded', function(){
  new Vue({
    el: '#header',
    // ......
  })
})
```
哇，很漂亮，可以正常用了。但是我没有发现的是，以前老代码老功能，**失效了！！**，WTF？？？这还是测试发现的，因为那个功能跟我做的需求不搭边（我做的登陆后的，登录前的功能在jq里，所以开发期间就没发现:sweat:）  

排查问题。通过二分调试法，确定问题，就是这个`new Vue`，为嘛呀。网上也没搜到这方面的问题。不过猜测可能是vue把他提取出来，一顿操作，然后再塞回去，所以原先绑定的事件就失效了，为了验证我的猜想，看源码呗。搜索关键字`appendChild`，找到
```javascript
function getOuterHTML (el) {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    var container = document.createElement('div');
    container.appendChild(el.cloneNode(true));
    return container.innerHTML
  }
}
```
再查找`getOuterHTML`函数调用处
```javascript
Vue.prototype.$mount = function (el, hydrating) {
  el = el && query(el);
  // ...
  var options = this.$options;
  if (!options.render) {
    var template = options.template;
    if (template) {
      // ......
    } else if (el) {
      template = getOuterHTML(el);
    }
    if (template) {
      var ref = compileToFunctions(template, {
        shouldDecodeNewlines: shouldDecodeNewlines,
        shouldDecodeNewlinesForHref: shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this);
      var render = ref.render;
      var staticRenderFns = ref.staticRenderFns;
      options.render = render;
      options.staticRenderFns = staticRenderFns;

    }
  }
  return mount.call(this, el, hydrating)
};
```
`mount`函数为
```javascript
Vue.prototype.$mount = function (
  el,
  hydrating
) {
  el = el && inBrowser ? query(el) : undefined;
  return mountComponent(this, el, hydrating)
};
```
然后`mountComponent`函数开始执行`beforeMount`钩子。按下不表

这个问题的主要地方就在于`getOuterHTML`函数中
```javascript
container.appendChild(el.cloneNode(true));
```
这一句。
记得在jquery中`$(el).clone(true)`会复制元素所有的事件处理，我以为原生的这块也是这样，查[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/cloneNode)却发现
![](https://upload-images.jianshu.io/upload_images/11264410-a1282b9f3a954e5d.png)
> **deep**(可选) 是否采用深度克隆,如果为true,则该节点的所有后代节点也都会被克隆,如果为false,则只克隆该节点本身

就是说原生`cloneNode`方法参数为`true`只是复制后代节点而已，也不复制事件。

至此，疑惑揭开了，。对应的解决办法当然是先执行`new Vue`后加载jq了。  
可以先遇到的`DOMContentLoaded`肯定是先加载啊。。。emmm...奥，出问题的那个事件没有在jq的`$(function(){})`内写着，怪不得先跑了

## 破案了

每次解决问题之后都觉得他是一个小问题，但是在遇到的时候很苦恼啊，定位问题，解决问题。都能学到不少，尤其是看源码的过程  

不得不说，Vue的源码写的真的漂亮
