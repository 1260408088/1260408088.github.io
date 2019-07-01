---
title: 使用nginx+tomcat实现动静分离
date: 2019-06-10 12:23:18
categories: nginx
tags: [配置,部署]
---

# 动态资源与静态资源的区别

微微的概括一下

- 静态资源： 当用户多次访问这个资源，资源的源代码永远不会改变的资源。

- 动态资源：当用户多次访问这个资源，资源的源代码可能会发送改变。

<!--more-->

# 什么是动静分离

动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路。

# 为什么要用动静分离

在我们的软件开发中，有些请求是需要后台处理的（如：.jsp,.do等等），有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件），这些不需要经过后台处理的文件称为静态文件，否则动态文件。因此我们后台处理忽略静态文件。这会有人又说那我后台忽略静态文件不就完了吗。当然这是可以的，但是这样后台的请求次数就明显增多了。在我们对资源的响应速度有要求的时候，我们应该使用这种动静分离的策略去解决。动静分离将网站静态资源（HTML，JavaScript，css，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问。这里我们将静态资源放到nginx中，动态资源转发到tomcat服务器中。因此，动态资源转发到tomcat服务器我们就使用到了前面讲到的反向代理了。

# 在nginx中的配置

``` xml
  ###静态资源访问
    server {
      listen       80;
      server_name  static.itmayiedu.com;
      location /static/imgs {
            root D:/; #会直接匹配D:下的static/imgs目录的 D:/static/imgs
		   index  index.html index.htm;
       }
    }
   ###动态资源访问
	 server {
      listen       80;
      server_name  www.kingstone.com; // 去hosts文件中配置

      location / {
         proxy_pass http://127.0.0.1:8080;
		 index  index.html index.htm;
       }
    }
```

# tip

图片这种静态资源，再次访问的话，会出现304状态码。这不是一种错误，而是对客户端有缓存情况下服务端的一种响应。

不明觉厉的解释：

``` xml
客户端在请求一个文件的时候，发现自己缓存的文件有 Last Modified ，那么在请求中会包含 If Modified Since ，这个时间就是缓存文件的 Last Modified 。因此，如果请求中包含 If Modified Since，就说明已经有缓存在客户端。服务端只要判断这个时间和当前请求的文件的修改时间就可以确定是返回 304 还是 200 。
对于静态文件，例如：CSS、图片，服务器会自动完成 Last
Modified 和 If Modified Since 的比较，完成缓存或者更新。但是对于动态页面，就是动态产生的页面，往往没有包含 Last Modified 信息，这样浏览器、网关等都不会做缓存，也就是在每次请求的时候都完成一个 200 的请求。
因此，对于动态页面做缓存加速，首先要在 Response 的
HTTP Header 中增加 Last Modified 定义，其次根据 Request 中的 If Modified Since 和被请求内容的更新时间来返回 200 或者 304 。虽然在返回
304 的时候已经做了一次数据库查询，但是可以避免接下来更多的数据库查询，并且没有返回页面内容而只是一个
HTTP Header，从而大大的降低带宽的消耗，对于用户的感觉也是提高
```



