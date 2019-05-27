---
title: springcloud动态网关
date: 2019-05-27 12:33:34
categories: springcloud
tags: [springcloud,配置] 
---

# ~~分布式配置~~

​	学以致用，之前学的分布式配置，这会有了用武之地了，将 zuul 的网关的配置放到码云上，方便管理，添加或者减少配置以后，不需要重启服务器就可以实现配置更新。之前的zuul的使用 {% post_link springCloud入门 %}，分布式配置说明{% post_link springCloud分布式配置中心 %}。一些配置就不在此处赘述了。（配置的没问题，就是没出现效果，急需的别往下看了）

<!--more-->

首先，在码云上项目中的config文件夹中创建一个 <span style="color:red">service-zuul-dev.yml </span>文件,文件内容如下：

``` yaml
### 配置网关反向代理    
zuul:
  routes:
    api-a:
     ### 以 /api-member/访问转发到会员服务
      path: /api-member/**
      serviceId: app-itmayiedu-member
    api-b:
        ### 以 /api-order/访问转发到订单服务
      path: /api-order/**
      serviceId: app-itmayiedu-order

```

然后在网关的项目中加上依赖文件：

``` xml
<!-- actuator监控中心 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- springcloud config 2.0 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>

```

网关项目的配置文件如下：

``` yaml

###服务注册地址
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8100/eureka/
###api网关端口号      
server:
  port: 8881
###网关名称  
spring:
  application:
    name: service-zuul #这个名称和码云上文件的前缀相同
  cloud:
    config:
    ####读取后缀
      profile: dev
      ####读取config-server注册地址
      discovery:
        service-id: config-server # 这是配置服务的项目ID如果不存在，
        enabled: true    
###默认服务读取eureka注册服务列表 默认间隔30秒

###开启所有监控中心接口
management:
  endpoints:
    web:
      exposure:
        include: "*"

```

需要在启动类中加上这个方法,这样才鞥读取到码云上的配置，和手动更新配置文件

``` java
// zuul配置能够使用config实现实时更新
	@RefreshScope
	@ConfigurationProperties("zuul")
	public ZuulProperties zuulProperties() {
		return new ZuulProperties();
	}
```

配置文件更新以后，方便生效。需要用 postman 来访问 http://127.0.0.1:8881/actuator/refresh 

效果：

这是一篇失败的博文，最后的结果让人头秃，我找了一个上午没找到原因在哪，后来试了别人的源码，开始还好好的，演示截图的时候出问题了！而且找不到