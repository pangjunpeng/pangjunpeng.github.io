---
title: 触发BFC加padding-top设为0.1px
date: 2018-03-27 18:32:37
tags: bug
---


 
这是上周博客还没搭好时的一个bug，具体找bug过程已经忘了，简单写一下记住。  

触发BFC可以加border可以加padding但是有时候会出现 加上会影响布局但是又不得不加 的情况，这个时候只需要`padding-top: .1px;`便可以了，完美！就这样吧（本来写了一大堆，还是简洁点吧）