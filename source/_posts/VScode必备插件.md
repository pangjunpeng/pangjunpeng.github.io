---
title: VScode必备插件
date: 2018-11-26 14:19:37
tags: 工具
---
换工作一个月了，创业公司每天忙得一批，好久都没有总结了。
项目是jsp+jq，糟心啊。。。当然这种低效的工作方式我是忍不了的。上手的第二天就着手引入vue，当然也碰到了很多问题。项目历史悠久，每次看代码感觉就像看历史书。这块以后另表  

先前是用webstorm开发，就是不想花时间找插件，配插件。这边都用vscode，为了方便沟通，也换过来吧。  

列几个我觉得不错的插件:
+ **GitLens**：git工具，可以很方便的在代码里看谁多会更改了
<!--more-->
+ **Bracket Pair Colorizer**，匹配括号（彩虹色），要匹配html的话得手动配置
```
"bracketPairColorizer.consecutivePairColors": [
        ["<", "</"],
        ["<", "/>"],
        [
            "Gold",
            "Orchid",
            "LightSkyBlue"
        ],
        "Red"
    ]
```
+ **autoprefixer**，fis3的autoprefixer不生效，只好先用这个凑合一段时候
+ react内JSX语法tab自动生成，在`setting.json`内加入`"emmet.triggerExpansionOnTab": true,`即可

就先这么多吧，个人是偏向实用性的，一些花里胡哨的就不放了，只是觉得这几个特别不错，尤其是这几个配置，怕忘了，以后发现好的再继续加  

+ **Comment Translate** 这可是个好东西，阅读源码过程中经常有大片的英文注释，有了这个插件后，鼠标放上去，就可以翻译出来了哈哈
