---
title: nginx(二)
date: 2019-06-07 19:21:35
categories: nginx
tags: [nginx,配置]
---

# nginx rewrite

​	Nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用。Rewrite主要的功能就是实现URL的重写，Nginx的Rewrite规则采用Pcre，perl兼容正则表达式的语法规则匹配，如果需要Nginx的Rewrite功能，在编译Nginx之前，需要编译安装PCRE库。通过Rewrite规则，可以实现规范的URL、根据变量来做URL转向及选择配置。

<!--more-->

#### Rewrite全局变量

​	nginx的rewrite规则就是使用正则匹配请求的url，然后根据定义的规则进行重写和改变，需ngx_http_rewrite_module模块来支持url重写功能，该模块是标准模块，默认已经安装。

| 变量              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| $args             | 这个变量等于请求行中的参数，同$query_string                  |
| $content length   | 请求头中的Content-length字段。                               |
| $content_type     | 请求头中的Content-Type字段。                                 |
| $document_root    | 当前请求在root指令中指定的值。                               |
| $host             | 请求主机头字段，否则为服务器名称。                           |
| $http_user_agent  | 客户端agent信息                                              |
| $http_cookie      | 客户端cookie信息                                             |
| $limit_rate       | 这个变量可以限制连接速率。                                   |
| $request_method   | 客户端请求的动作，通常为GET或POST。                          |
| $remote_addr      | 客户端的IP地址。                                             |
| $remote_port      | 客户端的端口。                                               |
| $remote_user      | 已经经过Auth Basic   Module验证的用户名。                    |
| $request_filename | 当前请求的文件路径，由root或alias指令与URI请求生成。         |
| $scheme           | HTTP方法（如http，https）。                                  |
| $server_protocol  | 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。                   |
| $server_addr      | 服务器地址，在完成一次系统调用后可以确定这个值。             |
| $server_name      | 服务器名称。                                                 |
| $server_port      | 请求到达服务器的端口号。                                     |
| $request_uri      | 包含请求参数的原始URI，不包含主机名，如”/foo/bar.php?arg=baz”。 |
| $uri              | 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。 |
| $document_uri     | 与$uri相同。                                                 |

## 实例demo

#### 判断IP地址来源

``` xml
 ## 如果访问的ip地址为192.168.5.165,则返回403
     if  ($remote_addr = 192.168.5.166) {  
         return 403;  
     }  
```

#### 限制浏览器访问

``` xml
## 不允许谷歌浏览器访问 如果是谷歌浏览器返回500
 if ($http_user_agent ~ Chrome) {   
         return 500;  
}
```



