---
title: Vue项目中使用全局less遇到的坑
date: 2018-04-09 09:54:16
tags: Q&A
---

最近在vue项目中加入了less，方法很简单，`npm install less less-loader --save`然后`webpack.base.conf.js`中加入
```javascript
{
  test  : /\.less$/,
  loader: "style-loader!css-loader!less-loader",
}
```
即可。   
但是`webstorm`中语法还是报错，在style标签上加入`type='text/less'`就好了。

然而项目中有好多组件样式可以复用，在`App.vue`中写的，由于作用域的原因其他组件并不能引入，会报undefined错误，只能另辟蹊径。  
<!-- more -->
+ 重新建一个`global.less`文件，然后在组建中单独`@import '../assets/css/global.less'`引入就可以用啦  
但是这样每个页面岂不是都要import一下！  
有没有什么办法可以一劳永逸呢，不得不说万能的网友还是厉害。

**方法如下**
1. 安装`sass-resources-loader`
2. 找到build文件夹下面的`utils.js`
3. 找到 `less: generateLoaders('less')  `
修改成
```
less: generateLoaders('less').concat({
    loader: 'sass-resources-loader',
    options: {
      resources: path.resolve(__dirname, '../src/assets/css/global.less')
    }
}), 
```
如果有多个文件，继续这个套路`concat`就可以了

### 这么看着爽多了 