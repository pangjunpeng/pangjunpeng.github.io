---
title: 解决vscode下默认node版本不对的问题，以及设置某个项目默认node版本
date: 2018-12-17 14:54:12
tags: bug
---
项目用的node版本为`4.5.0`，但是平时我也写一些个人的小项目，学习用，肯定是最新版本的node  

用[nvm](https://github.com/creationix/nvm)管理node版本。  

想安装什么版本，就
```bash
nvm install 4.5.0
```
在项目文件夹下，使用
```bash
nvm use 4.5.0
```
即可切换对应的node版本，很是方便。
也可设置默认版本
```bash
nvm alias default 11.2.0
```

**但是**，在VSCode中，就有问题了
<!--more-->

不管哪个项目，打开terminal之后都会提示
```
nvm is not compatible with the npm config "prefix" option: currently set to "/usr/local"
Run `npm config delete prefix` or `nvm use --delete-prefix v6.12.3 --silent` to unset it.
```
运行`node -v`发现，诶？总是`4.5.0`
每次打开项目都得`npm use 11.2.0`。  
虽然在`.zshsrc`里设置了`alias`，用`nl`命名`npm use 11.2.0`，但是每次都切换总不是个办法。  
于是谷歌，Stack Overflow，github上nvm和vscode的issue上找，真的是各有各的问题，各有各的方法。

总结一下，大概就这几点
+ 说有用的最多的一点：`nvm alias default x.x.x`设置默认版本可解决。------对我无效
+ 在项目对应的`launch.json`中加入对应版本的`runtimeExecutable` --------无效（就算有效也不能给每个项目都手动加这个呀，太麻烦）
```
{
    "type": "node",
    "request": "launch",
    "name": "App",
    "program": "${workspaceRoot}/index.js",
    "runtimeExecutable": "${env:HOME}/.nvm/versions/node/v6.9.2/bin/node"
    //或者 "runtimeExecutable": "11.2.0"
}

```
+ 在`~/.bash_profile`中加入`PATH="/usr/local/bin:$(getconf PATH)"`----无效
+ 还有说什么卸掉nvm的就不说了。。。（什么鬼）
+ 最后，在一个角落里，找到一个答案，他没有华丽的辞藻，也没有赞，有的只是一句
> Apparently the default shellArgs for osx are set to bash while I'm using zsh. I solved the problem by setting the shellArgs in my user settings to an empty array:
```
"terminal.integrated.shellArgs.osx": []

```
意思就是，你们都没解决我的问题，我用的zsh，在vscode的settings里加一句`"terminal.integrated.shellArgs.osx": []`就好了  

多么朴实，多么直达主题。我要登上我的Stack Overflow给他一个赞，然鹅
```
Thanks for the feedback! Votes cast by those with less than 15 reputation are recorded, but do not change the publicly displayed post score.
```
给他个评论吧
```
You must have 50 reputation to comment
```
好吧，我只能在心里默默感谢这位兄弟了。  
至于为什么这样能解决，我也是似懂非懂，好像是vscode默认用的bash，。。。但是我试了一下bash是正常的，而且terminal上明明写着zsh，烦恼~

## 为项目指定node版本，不必手动切换
vscode的node版本问题就解决了，那每次启动项目前还得切换版本，是不是很麻烦
推荐一个包叫做[avn](https://github.com/wbyoung/avn)，使用方法很简单
```
npm install -g avn avn-nvm avn-n
avn setup
```
然后在项目的根目录下新建一个文件，叫做`.node-version`，里面的内容就是你想要的node版本，这样以后cd进这个目录，那么node版本就会自动切到对应的版本，太方便有木有！

### 写完，干活！