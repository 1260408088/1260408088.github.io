---
title: springCloud分布式配置中心
date: 2019-05-24 16:57:05
categories: springcloud
tags: [rpc,配置]
---

# 1.分布式配置中心的作用

当一个系统中的配置文件发生改变的时候，我们需要重新启动该服务，才能使得新的配置文件生效，spring cloud config可以实现微服务中的所有系统的配置文件的统一管理，而且还可以实现当配置文件发生变化的时候，系统会自动更新获取新的配置。

<!--more-->

# 2.开始搭建

​	环境选择的是马云，呸呸呸“码云”（gitee）这一些列的注册什么的操作就不跳过了，反正中文的比github要方便的很多。码云上项目创建完毕后，先创建一个<span style="color:red">config</span>的文件夹，然后再这个添加一个文件，文件名就有讲究了我这用的是

<span style="color:red">server-config-dev.properties</span> server-config是前缀，这玩意一会有用的 dev 表示开发环境，真实开发中会有很多的配置文件 会有pro 等等的。配置文件里的内容就比较简陋了，重在说明问题......

``` properties
age=24 
```

补充一下：在码云创建的项目默认不是公开的，创建的时候注意一下，选择公开省去一些麻烦。

# 3.配置中心

​	首先你需要一个注册中心，不知道怎么配的，传送门在这 {% post_link  springCloud入门 %}。ok 这一步跳过

# 4.添加一个服务端

依赖文件：

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
    </parent>
    <!-- 管理依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.M7</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <!--spring-cloud 整合 config-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <!-- SpringBoot整合eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>
    <!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

配置文件：

``` yaml
###服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8888/eureka
spring:
  application:
    ####注册中心应用名称
    name: config-server
  cloud:
    config:
      server:
        git:
          ###git环境地址
          uri: https://gitee.com/kingpony/config.git
          ####搜索目录
          search-paths:
            - config
          # 码云上的项目你选择私有的话，需要加上
          #username: 
          #password:
      ####读取分支
      label: master
####端口号
server:
  port: 8889

```

启动主类：

``` java
@SpringBootApplication
@EnableConfigServer // 开启分布式配置中心服务器端
public class ServerApp {
    public static void main(String[] args) {
        SpringApplication.run(ServerApp.class, args);
    }
}
```

此时将项目运行起来，然后浏览器输入 localhost:8889/server-config-dev.properties 你就能看到

![1](1.png)

配置文件的全貌，如果你多放几个配置文件也是可以的换个名字读就行了

# 5添加一个客户端

依赖

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
    </parent>
    <!-- 管理依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.M7</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>
    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
        <!-- SpringBoot整合eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 开启手动刷新配置文件的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

    </dependencies>
    <!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

配置文件： bootstrap.yml

``` yaml
spring:
  application:
    ####注册中心应用名称,要和码云上所读取的那个配置文件的文件名前缀相同，不然读取不到的******
    name:  server-config
  cloud:
    config:
      ####读取后缀
      profile: dev
      ####读取config-server注册地址
      discovery:
        service-id: config-server
        enabled: true
#####    eureka服务注册地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8888/eureka
server:
  port: 8881
### 开启断点监控，不然刷新路径报错，起不到更新的作用
management:
  endpoints:
    web:
      exposure:
        include: "*"

```

启动类：(忽略下面无关配置，与方法.....)

``` java
@SpringBootApplication
@RestController
public class ClientApp {
        public static void main(String[] args) {
            SpringApplication.run(ClientApp.class, args);
        }

        @Value("${age}")
        String age;

        @RequestMapping(value = "/getUserName")
        public String getUserName () {
            return age;
        }
}
```

controller 

``` java
@RestController
@RefreshScope  // 手动刷新注解
public class AgeController {
    @Value("${age}")
    private String age;

    @RequestMapping("/getAge")
    public String getAge(){
        return age;
    }
}
```

访问 localhost:8881/getAge  效果如下

![2](2.png)

# 6.手动刷新配置文件

配置文件更改后还要启动客户端进行重新读取的话，这分布式配置中心的意义也就没有了，配置更新后，有两种方式可以进行刷新的

## 1.手动刷新

​	以上代码中有说明的，具体步骤

### 	1.首先需要给客户端添加一个依赖

``` xml
<!-- actuator监控中心 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2.客户端配置文件

添加以下文件

``` yaml
### 开启断点监控，不然刷新路径报错，没用
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

3.加上注解 

在接口类上添加，具体看上面

``` java
@RefreshScope
```

一顿操作以后，现在码云上将 age=24 改为 age=34

然后直接接着访问  localhost:8881/getAge  

![2](2.png)

结果没变化，不是浏览器的缓存的原因！你需要先访问一下这个地址刷新一下才可以

http://127.0.0.1:8881/actuator/refresh    postman的post方式访问，浏览器貌似没用的

然后就是阔以了 

![3](3.png)

## 2.实时刷新

没学会，先搁置，之后再更新