---
title: 使用redis来防止恶意请求
date: 2019-05-31 15:20:30
categories: redis
tags: [redis,拦截器]
---

# 奔入正题



redis 的一系列等等的，就不在这说了，以后完善笔记了再细细总结，公司的一个需求，想避免爬虫来爬网站。效果还没有试（自测是没问题），先来记录一哈。

<!--more-->

过程中使用的事springboot来搭建项目的，主要是方便，省去了那么多繁琐的配置。

## 还是惯例，先是依赖

``` xml
<dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.38</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>1.5.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>1.5.7.RELEASE</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

# 配置文件

打心底我比较喜欢properties文件的。

``` properties
#Redis
#spring.redis.host=127.0.0.1
redis.host=127.0.0.1
## Redis服务器连接端口
redis.port=6379
## 连接超时时间（毫秒）
redis.timeout=3000
## Redis服务器连接密码（默认为空）
redis.password=
## 连接池中的最大连接数
redis.poolMaxTotal=10
## 连接池中的最大空闲连接
redis.poolMaxIdle=10
## 连接池最大阻塞等待时间（使用负值表示没有限制）
redis.poolMaxWait=3000
```

redis的一个工具，用的是gitee的  <https://gitee.com/whvse/RedisUtil>  这个工具，还阔以就是设置key的时候不能顺带着把过期时间也给设置了，算美中不足的吧！（也可以在工具目录找到，有详细的说明）

先来微微的配置一下:

``` java
@Configuration
public class WebConfigurer implements WebMvcConfigurer { // 会重写一大堆方法，这就需要这一个
    
    @Autowired
    public DdosIntercepors Dintercepors; // 后面的拦截的配置，注入到这个里面才生效的
    
    public void addInterceptors(InterceptorRegistry registry) { // 拦截所有，过滤注册方法与登陆方法
        registry.addInterceptor(Dintercepors).addPathPatterns("/**").excludePathPatterns("/login", "/register");
    }
	// 拦截静态资源的方法
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

    }
}
```

下面这个才是真正的拦截配置：

``` java
@Component
public class DdosIntercepors implements HandlerInterceptor {
    @Autowired
    RedisUtil redisUtil;
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String ip = GetAddr.getIpAddress(request);
        if(redisUtil.sIsMember("ddos",ip)){ // 是否包含value
            returnJson(response,"请求太快，小心我美食九连，揉你猫猫皮"); //另一个同事测试的时候开玩笑的
            return false;
        }
        if(redisUtil.hasKey(ip)){ // 判断IP是否存在redis中
            int index = Integer.parseInt(redisUtil.get(ip))+1;
            redisUtil.setRange(ip,index+"",0);  // 更新一下当前ip访问的次数,这个方法不会覆盖过去时间
        }else{
            String start="1";
            redisUtil.setEx(ip,start,60, TimeUnit.SECONDS); // 每一个进来的ip都存入到redis，并设置过期时间
        }
        // 进行判断
        int count = Integer.parseInt(redisUtil.get(ip));
        if(count>=10){
            redisUtil.sAdd("ddos",ip);
            redisUtil.expire("ddos",60,TimeUnit.SECONDS);
        }
        System.out.println("velete");
        return true;
    }
	// 被拦截后给一个友好的提示（确实很友好）
    private void returnJson(HttpServletResponse response, String json) throws Exception{
        PrintWriter writer = null;
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=utf-8");
        try {
            writer = response.getWriter();
            writer.print(json);

        } catch (IOException e) {
            System.out.println("error");
        } finally {
            if (writer != null)
                writer.close();
        }
    }


    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

# controller层：

``` java
@RestController
public class UserController {
    @RequestMapping("/getuser")
    public List<String> getUser() {
        List<String> listUser = new ArrayList<String>();
        listUser.add("zhangsan");
        listUser.add("lisi");
        listUser.add("yushengjun");
        return listUser;
    }
}
```

# 拦截的效果：

![1](1.PNG)

补充说明redis的key设置过期时间后，如果你重新设置了key的值以后，过期时间会被覆盖就不存在了，但是list与set的添加元素的操作是不会的，使用时如果想更新值还不想失效时间被覆盖，建议使用 SETRANGE key [value] 来进行值的更新

``` shell
set key "3"  // 设置key为“3”
setrange key 0 "4" // 从第零位开始覆盖，我这里的更新数字是递增的，所以设置闷着头为 0 就可以了
```

