---
title: 彻底解决win10无线wifi自动掉线后必须重启的问题
date: 2018-03-28 11:07:44
tags: windows
---

作为一个win粉（曾经的，现在痴情mac），每次发布新版本都熬夜下载更新，但是win10发布之后，我还是乖乖地换回了8.1（win8.1史上最强不解释）。
win10从一出生到现在就一直在生产bug，我身边大多数win10选手都有 **wifi自动掉线，还连不上，必须重启** 的问题，但是网上文章千篇一律，都是什么 `设备管理器` - `允许计算机关闭此设备以节约电源`  
#### 根本没用好吧！！！
实在解决不了，只好等着微软更新了。

不过项目在银行，没有外网，只能自己开热点，一天能掉五六次，实在受不了了！！！

再次搜索之后发现知乎上有位大神摸索出来了，果然**时间可以治愈一切**

#### 解决办法如下
<!-- more -->
1. 右击 `网络` - `打开网络和共享中心`  
![](https://upload-images.jianshu.io/upload_images/11264410-fecccae36ae792dc.png)
2. 点击你当前网络的WLAN  
![](https://upload-images.jianshu.io/upload_images/11264410-83865d34435b1bb8.png)
3. 点击 `无线属性`  
![](https://upload-images.jianshu.io/upload_images/11264410-5bbed847188ada54.png)
4. 把`即使网络未广播其名称也连接`勾上  
![](https://upload-images.jianshu.io/upload_images/11264410-f88d8ba4658e8ffd.png)
5. 切换选项卡`安全`，点击`高级设置`  
![](https://upload-images.jianshu.io/upload_images/11264410-ab8e72968b64bc20.png)
6. 将`为此网络启用联邦信息处理标准(FIPS)兼容`勾上  
![](https://upload-images.jianshu.io/upload_images/11264410-e0210212dae11945.png)

### 完美解决！