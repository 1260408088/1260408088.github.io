---
title: springCloud配置文件解释
date: 2020-07-04 11:46:28
categories: [配置文件]
tags: [配置文件,springcloud]
---

``` yml
instance:
    prefer-ip-address: true # 就可以将IP注册到Eureka Server上，而如果不配置就是机器的主机名
    ip-address: 127.0.0.1 # 虽然prefer-ip-address 会自动获取ip注册到eureka中，自己再设置一下比较保险
    instance-id: ${spring.application.name}:${server.port}
```

对**instance-id**的一些解释，默认情况下不添加instance-id 会显示 ：

``` yaml
${spring.cloud.client.ipAddress}:${spring.application.name}:${spring.application.instance_id:${server.port}}
```

![](1.PNG)

``` yaml
${spring.application.name}:${server.port}
```

配置上以后，可以适当的删减显示

![](2.PNG)

// TODO

