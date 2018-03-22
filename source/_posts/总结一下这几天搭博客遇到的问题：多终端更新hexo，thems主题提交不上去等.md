---
title: 总结一下这几天搭博客遇到的问题：多终端更新hexo，thems push不上去等
date: 2018-03-22 15:21:24
tags: hexo
---
因为以前搭过hexo，所以这次速度快了许多，稍微熟悉一下，申请github pages，npm install hexo，hexo init等等一气呵成，一下午也就搞定了。

**但是！**（再也不用editor.md了（捂脸哭）码了半天一个重启...说好的保存呢啊摔！！！）家里公司两台电脑啊，两端同步操作就懵逼了，而且对刚买的mac还不熟悉...  
![](https://upload-images.jianshu.io/upload_images/11264410-85c864b78cff126e.jpg)
其实也可以写好md传到云盘，然后到家down下来发布。。。  
<img src="https://upload-images.jianshu.io/upload_images/11264410-febcbec4a270819f.jpg" width="100"/>
**emmm......我！就！不！！！  
闹心，他不应该是这个样子(╯‵□′)╯︵┻━┻**

经过一番搜索后，我发现.........(*╹▽╹*)
<!-- more -->
`google`和`Stack Overflow`真是太好用了(づ｡◕‿‿◕｡)づ撒花~， 什么渣渣百度，哼~

---
好了不说废话了，开始干活~

**假定已在一台电脑上安装好hexo并且hexo d了**

+ **首先来说说hexo的原理：**
    * 编写markdown
    * `hexo clean` 清理你的项目缓存
    * `hexo g` 将你刚刚编写的md编译为浏览器可以识别的html
    * `hexo d` 发布到远程仓库

+ **好了，了解了hexo基本原理之后，就有思路了。**
    + hexo d是发布到`_config.yml`中配置的branch分支上（一般都是master主分支）
    + 所以就可以将hexo主文件（未编译的）放到另一个仓库中，更新文章时，只需再将这个仓库提交一下就可以了
    + 而`git clone`下来都是xxx.github.io仓库，所以就可以建一个分支来存储，这样，平时写md，更新等等所有操作，都只需要在此条分支上进行即可。

+ **开始**
    1. `git branch hexo`，新建分支`hexo`。
    2. `git checkout hexo`，切换到分支`hexo`。
    2. `git add commit等正常操作`，因为hexo带.gitignore，所以不用担心将无用东西传上去。
    3. `git push origin hexo`，**注意是push到hexo分支**。
    4. 至此，hexo主站程序就上传到`hexo`分支了
    5. 发布文章时，正常流程写md，`hexo clean`，`hexo g -d`就可以啦~

+ **另一台电脑上**
    + `git clone ......` 克隆下来
    + **注意**，这个时候克隆下来你看到的和github上master分支上一样，找不到source
    + 所以就需要`git checkout hexo`切换至hexo分支
    + **注意**，这个时候你clone下来的，没有node_modules，所以得在hexo分支上`npm install hexo --save`一下
    + 然后就可以正常操作啦~
+ **每天换电脑时，记得`git pull origin hexo`哦~**

---
因为平时工作使用svn，所以话费了点时间在学习git上，其实安装hexo只需要知道那几个常用的就可以了
## 遇到的问题
* **每次push都会弹出来让输入username和password**
    * **问题所在**：采用了了https方式提交代码。可以`git remote -v`查看
    * **解决办法**：`git remote rm origin`删除远程连接，然后
    `git remote add origin git@github.com:(用户名)/版本库名`重新使用ssh方式连接，切换方式在github的clone那块可以选择
* 最大的一个问题，hexo s时No layout没有渲染，打开页面空白
    * **问题所在**：由于themes中yilia时clone的，所以不在我的版本库中，自然也就push不上去
    * **解决办法**，删除.git文件夹，重新add commit push
    * 继而又引发一个**问题**fatal: Pathspec ‘themes/yilia/_config.yml’ is in submodule ‘themes/yilia’ ，是`yilia`下没有.git所致
    * **解决办法**：
    ```
    git rm -rf --cached themes/yilia
    git add themes/yilia/*
    git commit
    git push
    ```
    * **至此，完美解决！**
    
#### 这个过程中遇到的问题肯定不止这两个，其他的通过`Google`/`Stack Overflow`都可以轻易解决（再次嫌弃百度一次(╯‵□′)╯︵┻━┻）
#### 这几天过程中配置了`iterm`，安装`brew`过程中也遇到了权限问题等等等等，`stackoverflow`真是个好东西，百度根本找不到解决办法。学会了`vim`的基本使用
开心~，继续加油吧！
