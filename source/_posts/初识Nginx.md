---
title: 初识Nginx
date: 2019-05-28 17:34:53
categories: nginx
tags: [服务器, 反向代理]
---

# 应用场景

- http服务器，使用nginx做静态服务器、图片服务器
- 虚拟主机配置，将一台服务器拆分成多个网站部署
- 反向代理，可以隐藏真实的 ip 访问地址
- 配建接口网关（解决跨域问题）
- 实现网站的动静分离
- 防止DDos，防盗链
- 配置缓存

<!--more-->

# 配置文件结构说明

仅仅有server，后面再进行补充

``` xml
# 内部创建服务器、监听端口
server {
	    #server 监听的端口号
        listen       80;
	    #服务器name 配置域名
        server_name  solo.ning.com;
		#匹配所有url地址
        location / {
			# 监听拦截后 跳转根目录 资源目录文件 html文件
			root html
			# 默认首页 index.html
			index index.html index.htm;
        }
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

# Nginx虚拟主机配置

1、基于域名的虚拟主机，通过域名来区分虚拟主机——应用：外部网站

2、基于端口的虚拟主机，通过端口来区分虚拟主机——应用：公司内部网站，外部网站的管理后台

3、基于ip的虚拟主机，几乎不用。

## 基于虚拟主机配置域名

在hosts文件中添加 bbs.ning.com、www.ning.com 都映射到127.0.0.1上（下面默认，更改的话，会有说明）

``` xml
 server {
        listen       80;
        server_name  bbs.ning.com;
        location / {
			root data/bbs; # 在根目录中创建，index的内容就一<h1>BBS</h1>
			index index.html index.htm;
        }
	}

 server {
        listen       80;
        server_name  www.ning.com;
        location / {
			root data/www; # 在根目录中创建，index的内容就一<h1>www</h1>
			index index.html index.htm;
        }
}
```

效果分别为

![www](1.PNG)

![BBS](2.PNG)

## 基于端口的虚拟主机

```xml
server {
        listen       8080;
        server_name  bbs.ning.com;
        location / {
			root data/bbs; # 在根目录中创建，index的内容就一<h1>BBS</h1>
			index index.html index.htm;
        }
	}

 server {
        listen       8081;
        server_name  bbs.ning.com; #此处重复没问题
        location / {
			root data/www; # 在根目录中创建，index的内容就一<h1>www</h1>
			index index.html index.htm;
        }
}
```

效果：

![3](3.PNG)

![4](4.PNG)

