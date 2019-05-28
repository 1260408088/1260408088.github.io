---
title: swaggerr的使用
date: 2019-05-27 22:40:33
categories: swagger
tags: 接口文档
---

# 传统文档的痛点

- 对API文档进行更新的时候，需要通知前端开发人员，导致文档更新交流不及时；

- API接口返回信息不明确

- 大公司中肯定会有专门文档服务器对接口文档进行更新。

- 缺乏在线接口测试，通常需要使用相应的API测试工具，比如postman、SoapUI等

- 接口文档太多，不便于管理

<!--more-->

# Swagger具有以下优点

- 功能丰富：支持多种注解，自动生成接口文档界面，支持在界面测试API接口功能；

- 及时更新：开发过程中花一点写注释的时间，就可以及时的更新API文档，省心省力；

- 整合简单：通过添加 pom 依赖和简单配置，内嵌于应用中就可同时发布API接口文档界面，不需要部署独立服务。

# 使用springboot的集成

后面会又springCloud的和zuul整合进行文档的管理

依赖文件

``` xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
    </parent>

    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringBoot swagger2 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.8.0</version>
        </dependency>
        <!--SpringBoot swagger2 -UI -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.8.0</version>
        </dependency>
    </dependencies>
```

文件的配置类（java形式的）

``` java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
                // 自行修改为自己的包路径
                .apis(RequestHandlerSelectors.basePackage("com.king.swagerr.controller")).paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title("api文档").description("restfun 风格接口")
                // 服务条款网址
                // .termsOfServiceUrl("http://blog.csdn.net/forezp")
                .version("1.0")
                // .contact(new Contact("帅呆了", "url", "email"))
                .build();
    }
}
```

接口类

``` java
@RestController
@RequestMapping("api")
@Api("swaggerDemoController相关的api")
public class SwaggerDemoController {

    private static final Logger logger = LoggerFactory.getLogger(SwaggerDemoController.class);

    @ApiOperation(value = "根据id查询学生信息", notes = "查询数据库中某个的学生信息")
    @ApiImplicitParam(name = "id", value = "学生ID", paramType = "path", required = true, dataType = "Integer")
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)  // 必须声明请求方法，负责会生成好多个无用说明
    public String getStudent(@PathVariable int id) {
        logger.info("开始查询某个学生信息");
        return "success";
    }

}
```

访问地址  <http://localhost:8080//swagger-ui.html#/>

# zuul网关整合Swagger

zuulServer 网关的配置，其他的配置忽略了

``` yaml
zuul:
  routes:
    api-a:
      path: /api-member/**
      service-id: service-member
    api-b:
      path: /api-order/**
      service-id: service-order
```

依赖文件,统一都用这一个

``` xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

zuulServer的Swagger的配置：

``` java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("分布式购物系统")
                .description("购物系统接口文档说明")
                .termsOfServiceUrl("http://localhost:8081")
                .contact(new Contact("guoning", "", "1260408088@qq.com"))
                .version("1.0")
                .build();
    }
}

```

orderServer与userServer配置相同：

``` java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.order.controller")) // 扫描包
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() { // 下面的信息，自行更改
        return new ApiInfoBuilder()
                .title("购物系统-订单模块")
                .description("购物系统订单模块接口文档说明")
                .termsOfServiceUrl("http://localhost:8083")
                .contact(new Contact("guoning", "", "1260408088@qq.com"))
                .version("1.0")
                .build();
    }
}
```

controller中的配置,

``` java
@RestController
@RequestMapping("api")
@Api("swaggerDemoController相关的api")
public class SwaggerDemoController {
	
    private static final Logger logger = LoggerFactory.getLogger(SwaggerDemoController.class);

    @ApiOperation(value = "根据id查询学生信息", notes = "查询数据库中某个的学生信息") // 描述
    @ApiImplicitParam(name = "id", value = "学生ID", paramType = "path",required = true, dataType = "Integer") // 参数描述
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)  // 必须声明请求方法，负责会生成好多个无用说明
    public String getStudent(@PathVariable int id) {
        logger.info("开始查询某个学生信息");
        return "success";
    }

}
```

就到这了，其中有参考了 <https://www.jianshu.com/p/af4ff19afa04> ，因为课程视频中不太详细。