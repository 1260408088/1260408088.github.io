---
title: springCloud入门
date: 2019-05-24 11:50:07
categories: springcloud
tags: rpc
---

# 1.Springcloud的模块

- rest、feign 客户端调用工具
- ribbon 负载均衡 
- zuul 接口网关
- euraka 服务注册中心
- Hystrix 熔断器  

<!--more-->

# 2.注册中心的使用

直接上代码，不逼逼。简单的演示一下服务的注册与使用

## 服务提供者

依赖文件：

``` xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Dalston.RC1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
    </dependencyManagement>

    <build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
    </build>

    <repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    </repositories>
```

配置文件：

``` yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/
server:
  port: 8762   # 本程序的端口
spring:
  application:
    name: service-member  # 本程序在注册中心的名称,提供给消费者使用
```

突然想起来顺序是不是有问题，注册中心被我给吃了！

启动类：(偷懒了，启动类要与package平级，这样可以省去写包扫描的注解)

![0](0.png)

``` java
@SpringBootApplication
@EnableEurekaClient   // 注册中心客户端
public class UserApp {

    public static void main(String[] args) {
        SpringApplication.run(UserApp.class, args);
    }

}
```

写一个假的controller略表意思

``` java
@RestController
public class UserController {
    @Value("${server.port}")   // 直接可以读取配置文件的内容，后序做负载均衡需要用上
    private String port;
    @RequestMapping("/getuser")
    public List<String> getUser() {
        List<String> listUser = new ArrayList<String>();
        listUser.add("zhangsan");
        listUser.add("lisi");
        listUser.add("yushengjun");
        listUser.add(port);
        return listUser;
    }
}
```

赶紧把注册中心给补上..... 

## 注册中心

依赖文件:

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--eureka server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <!-- spring boot test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

配置文件：

``` yaml
server:
  port: 8888
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ # eureka注册中心的地址

```

注册中心的主类：

``` java
@SpringBootApplication
@EnableEurekaServer 
public class Launcher {
    public static void main(String[] args) {
        SpringApplication.run(Launcher.class, args);
    }
}
```

## 服务消费者

依赖文件：

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

配置文件：

``` yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/
server:
  port: 8764
spring:
  application:
    name: service-order
```

启动主类：

``` java
@SpringBootApplication
@EnableEurekaClient
public class OrderApp {
    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }

    @Bean
    @LoadBalanced  // 使用rest方式的时候，进行负载均衡时使用 (开启ribbon)
    RestTemplate restTemplate() {   
        return new RestTemplate();
    }

}
```

虚假的controller：

``` java
@RestController
public class OrderController {
    @Autowired
    private MemberService memberService;

    @RequestMapping("/getOrderUserAll")
    public List<String> getOrderUserAll() {
        System.out.println("订单服务开始调用会员服务");
        return memberService.getOrderByUserList();
    }

}
```

虚假的service：

``` java
@Service
public class MemberService {
    @Autowired
    RestTemplate restTemplate;
    public List<String> getOrderByUserList() {
        return restTemplate.getForObject("http://service-member/getuser", List.class);
    }
}
```

## 运行状态

先启动注册中心，再启动消费者与提供者，负责会出错的....

注册中心运行后的状态：

![1](1.png)

两个服务，服务名称对应各自的配置文件中的名称。

先访问服务的提供者，直接通过地址访问 localhost:8762/getuser

![2](2.png)

然后通过消费者内部调用服务的方式，进行访问 http://localhost:8764/getOrderUserAll

![3](3.png)

此处因为开启了负载均衡，访问的时候会一替一下的（轮训的算法）404，解决方案

	1.再开启一个服务的提供者，再idea下开启需要先讲配置文件中的端口改一下，然后如图勾选上并行运行就阔以了！

![4](4.png)

2.或者去掉负载均衡的注解，安心的做个咸鱼~~~~

# 3.Feign

上面介绍了的rest方式调用服务外，还存在着这一种调用的方式（已经整合了ribbon，自带负载均衡）

废话少说，依赖文件

``` xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

配置文件：

``` yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/
server:
  port: 8765
  tomcat:
    max-threads: 50
spring:
  application:
    name: service-order-feign
# 开启熔断，后面说
#feign:   
#   hystrix:
#     enabled: true
#hystrix:
#   command: 
#     default: 
#       execution: 
#        isolation:
#         thread: 
#          timeoutInMilliseconds: 4000
```

主类

``` java
@SpringBootApplication
@EnableEurekaClient 
@EnableFeignClients // 开启feign的客户端
public class Fegincon {
    public static void main(String[] args) {
        SpringApplication.run(Fegincon.class, args);
    }

}
```

service接口 (Feign 采用的是基于接口的注解)

``` java
@FeignClient(value = "service-member")
public interface MemberFeign {
	@RequestMapping("/getuser")
	public List<String> getOrderByUserList();
}
```

一个假的controller

``` java
@RestController
public class FeignMemberController {
	@Autowired
	private MemberFeign memberFeign;
	@RequestMapping("/getFeignOrderByUserList")
	public List<String> getFeignOrderByUserList() {
		return memberFeign.getOrderByUserList();
	}
	@RequestMapping("/getOrderFeign")
	public String getOrderFeign() {
		return "getOrderFeign";
	}
}
```

效果如图：（比较智能，没有404，知道我没有多开启项目，没有报404）

![5](5.png)

# 4.zuul网关

顾名思义，编不下去了......自己查去吧<www.baidu.com>

依赖文件

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

配置文件

``` yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/
server:
  port: 8769
spring:
  application:
    name: service-zuul
zuul:
  routes:
    api-a:
      path: /api-member/**    # 映射那么一下，和nginx的赶脚一样
      service-id: service-member
    api-b:
      path: /api-order/**
      service-id: service-order
```

启动主类

``` java
@EnableZuulProxy  // 开启网关代理
@EnableEurekaClient
@SpringBootApplication
public class ZuulApp {

    public static void main(String[] args) {
        SpringApplication.run(ZuulApp.class, args);
    }

}
```

此处演示的是一个过滤器，必须让请求携带一些参数负责会给报错

过滤器代码如下：

``` java
@Component
public class MyFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    public boolean shouldFilter() {
        return true;
    }

    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        if (accessToken != null) {
            return null;  // 拦截与否都会返回null
        }
        log.warn("token is empty");
        ctx.setSendZuulResponse(false);
        ctx.setResponseStatusCode(401);
        try {
            ctx.getResponse().getWriter().write("token is empty");
        } catch (Exception e) {
        }
        return null;

    }
}
```

效果

	未带token参数

![6](6.png)

	带了token参数

![7](7.png)

不加“api-member”也是可以访问的，但是没有过滤token的功能

## 获得客户端信息的接口

DiscoveryClient 可以获得服务中的详细信息，端口、url、名称等.....

``` java
@Autowired
private DiscoveryClient discoveryClient;
@RequestMapping("/getClient")
    public void getClent(){
        List<ServiceInstance> instances = discoveryClient.getInstances("service-order");
        for (ServiceInstance server:instances) {
            System.out.println(server.getUri());
        }
    }
```

输出如下：

![8](8.png)