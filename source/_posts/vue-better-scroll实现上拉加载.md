---
title: vue+better-scroll实现上拉加载
date: 2018-07-06 16:58:50
tags: 
- vue
- js
---
需求中有拖动，就想到better-scroll，仔细一看文档，发现好多api我那个[selectPicker](https://github.com/pangjunpeng/myWheels/tree/master/src/components/SelectPicker)中也有，果然广大程序猿思路都是一样的，只是可能我写的不够优雅吧，嘻嘻

因为内部至少需要一个元素，所以如果有ajax异步请求，将下面代码放入this.$nextTick()中
```javascript
this.scroll = new BScroll(this.$refs.xxx)
```
就可以拉动啦，需要注意的是，this.$refs.xxx下面必须要有一层wrapper包裹，跟我[selectPicker](https://github.com/pangjunpeng/myWheels/tree/master/src/components/SelectPicker)原理一模一样，但是我可木有抄呀

其他配置项在第二个参数中配置
```javascript
this.scroll = new BScroll(this.$refs.xxx, {
	click          : true, //可点击
	scrollY        : true, // y轴方向可滑动
	probeType      : 2, // 手指滑动时派发scroll事件
	bounce         : {
	  top: false // 顶部无阻尼效果
	},
})
```
本来想用pullup来做上拉的，但是下滑速度过快，就会导致滑动距离超出`threshold`进而加载了下一页，设置swiperBounceTime总感觉不理想，于是就自己用scroll事件写  
<!--more-->
`this.scrollEl.maxScrollY`用来获取scroll 最大纵向滚动位置，超过80px变换文字
```javascript
this.scrollEl.on('scroll', res => {
	if((res.y + 80) < this.scrollEl.maxScrollY){
	  this.imgSty.transform =  'rotate(180deg)'
	  this.imgSrc = './img/arrow.png'
	  this.loadTips = '释放加载'
	}else{
	  this.imgSty.transform = 'rotate(0deg)'
	  this.imgSrc = './img/arrow.png'
	  this.loadTips = '上拉加载更多...'
	}
})
```
释放之后再加载，**判断释放位置**是否大于80
```javascript
this.scrollEl.on('touchEnd', res => {
  if((res.y + 80) < this.scrollEl.maxScrollY ){
	this.pageNum++
	this.getList()
	this.imgSrc = './img/loading.png'
	this.loadTips = '正在加载'
  }
})
```

然后，出现了个问题，每次加载完下一页数据后页面都返回顶部了，仔细一看，原来是每次getList之后都会new BScroll才导致的，简单

`this.scroll.refresh()`用来`重新计算 better-scroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常。`  
判断是否已有this.scroll,如果有，就refresh，代码：
```javascript
self.$nextTick(function () {
  if(!this.scrollEl){
	this.scrollEl = new BScroll(this.$refs.listWrapper, {
	  click          : true,
	  scrollY        : true,
	  probeType: 2,
	  bounce         : {
		top: false
	  },
	})
	this.scrollEl.on('scroll', res => {
	  if(!this.hasEnd){
		if((res.y + 80) < this.scrollEl.maxScrollY){
		  this.imgSty.transform =  'rotate(180deg)'
		  this.imgSrc = './img/arrow.png'
		  this.loadTips = '释放加载'
		}else{
		  this.imgSty.transform = 'rotate(0deg)'
		  this.imgSrc = './img/arrow.png'
		  this.loadTips = '上拉加载更多...'
		}
	  }
	})
	this.scrollEl.on('touchEnd', res => {
	  if(!this.hasEnd && (res.y + 80) < this.scrollEl.maxScrollY ){
		this.pageNum++
		this.getList()
		this.imgSrc = './img/loading.png'
		this.loadTips = '正在加载'
	  }
	})
  }else {
	this.scrollEl.refresh()
  }
})
```
`this.hasEnd`用来判断是否没有数据，其实我觉得没有数据之后应该移除上拉事件，但是，emmm...赶工，你懂得

2018年7月18日12:16:44
哎呀，移出上拉也不好，万一突然就有数据了呢，虽说用户刷新也不是什么难事，但是他不知道哇  
嗯！就这样吧