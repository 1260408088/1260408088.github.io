---
title: nginx(一)copy
date: 2019-06-07 13:52:26
categories: nginx
tags: [负载均衡,nginx]
---

# 负载均衡的几种算法

- 轮询机制
- 权重
- IP绑定
- faiil
- url

## 	tip

在nginx1.9之前nginx指支持七层负载均衡，应用层的负载均衡（所谓七层，物理层、数据链路层、网络层、传输层、会话层、表示层、应用层、）在1.9之后支持了四层传输协议也就是网络层的负载均衡。本质上时支持那个层的协议。

# 负载均衡的配置

``` xml
#上游服务器 
upstream  backServer{
	    server 127.0.0.1:8080;
	    server 127.0.0.1:8081;
	}
 	
	server {
        listen       80;
        server_name  www.itmayiedu.com;
        location / {
		    ### 指定上游服务器负载均衡服务器
		    proxy_pass http://backServer;
            index  index.html index.htm;
        }
    }

```

## 带权重的配置

``` xml
upstream  backServer{
	server 127.0.0.1:8080 weight=1;
	server 127.0.0.1:8081 weight=2;
	}
 	
	server {
        listen       80;
        server_name  www.itmayiedu.com;
        location / {
		    ### 指定上游服务器负载均衡服务器
		 proxy_pass http://backServer;
            index  index.html index.htm;
        }
    }

```

## Ip绑定

每个请求按访问IP的哈希结果分配，使来自同一个IP的访客固定访问一台后端服务器，并且可以有效解决动态网页存在的session共享问题。俗称IP绑定。

``` xml
upstream  backServer{
	    server 127.0.0.1:8080 ;
		server 127.0.0.1:8081 ;
		ip_hash; 
	}
 	
	server {
        listen       80;
        server_name  www.itmayiedu.com;
        location / {
		    ### 指定上游服务器负载均衡服务器
		    proxy_pass http://backServer;
            index  index.html index.htm;
        }
    }

```

## 故障转移

当上游服务器(真实访问服务器),一旦出现故障或者是没有及时相应的话，应该直接轮训到下一台服务器，保证服务器的高可用。

``` xml
server {
        listen       80;
        server_name  www.itmayiedu.com;
        location / {
		### 指定上游服务器负载均衡服务器
		proxy_pass http://backServer;
	    ###nginx与上游服务器(真实访问的服务器)超时时间 后端服务器连接的超时时间_发起握手等候响应超时时间
		proxy_connect_timeout 1s;
		###nginx发送给上游服务器(真实访问的服务器)超时时间
         proxy_send_timeout 1s;
		### nginx接受上游服务器(真实访问的服务器)超时时间
         proxy_read_timeout 1s;
         index  index.html index.htm;
        }
    }
```



