---
layout: w
title: 给hexo博客的NEXT添加一个云日历
date: 2019-06-24 17:48:08
categories: 工具
tags: 工具,突发奇想
---

# 一点废话

hexo中有文件的归档，但是博文的数目多了，浏览的时候也是很不方便的。于是我就有找个云日历的想法了，折腾了几天，网上的方法都试过了。但是没出效果。于是想着自己来写一个。这自己写的这部分是基于净土大神的日历插件。也是我这个云日历的灵魂，感谢大神。

<!---more-->

# 效果

先看效果，不满意就不用向下看了。可以去找找其他的。图片什么的我就不截了，你直接去我的博客中看吧！

[会飞的扫帚](https://1260408088.github.io/)还是放一张图，吸引一下别人吧！

![](1.PNG)

![](2.PNG)

![](3.PNG)

# 进入正题

## 	插件

​	先贴上净土大神写的日历插件，直接在在命令行窗口安装。前提是你要装的有node.js

``` shell
npm install --save git://github.com/howiefh/hexo-generator-calendar.git
```

安装完毕以后，运行一下 <span style="background-color:black;color:white" >hexo g </span>，然后去hexo目录下的public 目录下看看是是否有一个calendar.json 文件，这个很重要的。

## 	一点说明

我使用的是Next的muse主题，比较简约，而且有一个空间比较大的侧边栏。其他的主题，你们自己尝试吧！

## 	文件准备

放到百度云盘里自取，这是[地址](https://pan.baidu.com/s/1X8hXqkVePXkWc1oGNLnf-w) (wl8h)，之前我将这三个引入index.html的文件放在github上引用，后来GitHub的策略便了，我就放到next主题的文件夹内了。

![](4.4.PNG)

## 开始整合

​	找到 hexo\themes\next\layout\_custom\siderbar.swig 文件，将准备好的index.html文件同级别放置，然后打开sidebar.swig文件。在最上面添加代码：

``` html
<div id="coustomerCal">
	{% include "index.html" %}  // swig的语法，我是个菜鸡我也是查资料才知道的
</div>
```

找到hexo\themes\next\source 目录，将对应的css文件，js文件放入到对应的目录下即可

![](5.5.PNG)

## 最后

​	最开始的那个日历插件，如果没有问题的话，会出现在你的hexo仓库的根目录下，你在github上打开，点击row，copy网址，然后替换**calendar.js**最后的地址,文件直接拉到最后就看到了。下面的地址是我自己的，你要替换成你的。

![](6.6.PNG)

## 尾记

​	这个玩意里面可优化的东西很多，如果你使用的话，你自己优化一下，我因为不是专业的前端，也秉持着能将就就将就的原则也没改，这篇文章的修改还是因为github的策略有变我才写的。如果有问题，请留言，我看到了一定会回复的！

