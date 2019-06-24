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

放到百度云盘里自取，这是[地址](https://pan.baidu.com/s/1pGw6g1AYQE-b_Wj8Hj1LPA) (3n3u)，下载后我建议，放到你自己的github上的项目中，便于引用，我特别创建了一个static仓库就用来存放这些静态的文件。科普一下github上的js、css文件如何来引用。

![4](4.PNG)

点击raw，后这样的页面是不是似曾相识

![](5.PNG)

直接使用 这个地址是不行的，需要把 githubusercontent 改为githack 才可以,例如下面

![](6.PNG)

至此准备工作基本结束了。

## 开始整合

找到 hexo\themes\next\layout\_custom\siderbar.swig 文件，将准备好的index.html文件同级别放置，然后打开sidebar.swig文件。在最上面添加代码：

``` html
<div id="coustomerCal">
	{% include "index.html" %}  // swig的语法，我是个菜鸡我也是查资料才知道的
</div>
```

然后你需要修改index中的资源的引用，将 css、js文件都改为你自己的静态资源引用，当然了，你使用我的也可以，但是后期我修改的时候，可能会造成问题，所以建议你自己添加。大致就是这样了，如果过程中遇到问题，可以留言！最后附上我的github中的博客仓库，出现问题您可以参考一下！[hexo仓库](https://github.com/1260408088/1260408088.github.io)

