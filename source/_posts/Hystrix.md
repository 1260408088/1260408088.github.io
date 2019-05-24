---
title: Hystrix
date: 2019-05-23 13:28:20
categories: springcloud
tags: [分布式,rpc]
---

# 1.服务器雪崩

在高并发的情况下，所有的服务器线程都在处理高并发下的请求，倒置其他请求无法处理。A服务请求B服务，tomcat默认并发是50个，20000个请求一起过来时，B服务都在处理A服务的请求，其他请求（直接访问B服务的请求）<span style="color:red">都在等待状态！</span>

<!--more-->

# 2.几个概念

## 服务降级

在高并发情况下，防止用户一直等待，使用服务降级方式(直接返回一个友好的提示给客户端，调用fallBack方法）

## 服务熔断

熔断机制目的为了保护服务，在高并发的情况下，如果请求达到一定极限(可以自己设置阔值)如果流量超出了设置阈值，让后直接拒绝访问，保护当前服务。使用服务降级方式返回一个友好提示，服务熔断和服务降级一起使用。

## 服务隔离

因为默认情况下，只有一个线程池会维护所有的服务接口，如果大量的请求访问同一个接口，达到tomcat 线程池默认极限，可能会导致其他服务无法访问。解决服务雪崩效应:使用服务隔离机制(线程池方式和信号量)，使用线程池方式实現隔离的原理:  相当于每个接口(服务)都有自己独立的线程池，因为每个线程池互不影响，这样的话就可以解决服务雪崩效应。

###   线程池隔离:

每个服务接口，都有自己独立的线程池，每个线程池互不影响。

###   信号量隔离:

使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，当请求进来时先判断计数器的数值，若超过设置的最大线程个数则拒绝该请求，若不超过则通行，这时候计数器+1，请求返 回成功后计数器-1。

此处应该有demo

配置文件：

``` yaml
feign:
  hystrix:
enabled: true
    
#### hystrix禁止服务超时时间
hystrix:  
 command: 
   default: 
      execution: 
       timeout: 
        enabled: false


```

依赖文件：

``` xml
	<!-- hystrix断路器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
```



``` java
@RestController
public class OrderServiceImpl extends BaseApiService implements IOrderService {  // 继承与接口不用细看，使用feigin整合
	// 订单服务继承会员服务接口，用来实现feign客户端 减少重复接口代码
	@Autowired
	private MemberServiceFeigin memberServiceFeigin;

	@RequestMapping("/orderToMember")
	public String orderToMember(String name) {
		UserEntity user = memberServiceFeigin.getMember(name);
		return user == null ? "没有找到用户信息" : user.toString();
	}

	// 没有解决服务雪崩效应
	@RequestMapping("/orderToMemberUserInfo")
	public ResponseBase orderToMemberUserInfo() {
		return memberServiceFeigin.getUserInfo();
	}

	// 解决服务雪崩效应
	// fallbackMethod 方法的作用：服务降级执行
	// @HystrixCommand 默认开启线程池隔离方式,服务降级,服务熔断
	// 设置Hystrix服务超时时间
	/**
	 * @HystrixCommand<br>
	 * 					默认开启服务隔离方式 以线程池方式<br>
	 *                     默认开启服务降级执行方法orderToMemberUserInfoHystrixFallback<br>
	 *                     默认开启服务熔断机制<br>
	 * 
	 * @return
	 */
	@HystrixCommand(fallbackMethod = "orderToMemberUserInfoHystrixFallback")
	@RequestMapping("/orderToMemberUserInfoHystrix")
	public ResponseBase orderToMemberUserInfoHystrix() {
		System.out.println("orderToMemberUserInfoHystrix:" + "线程池名称:" + Thread.currentThread().getName());
		return memberServiceFeigin.getUserInfo();
	}

	public ResponseBase orderToMemberUserInfoHystrixFallback() {
		return setResultSuccess("返回一个友好的提示：服务降级,服务器忙，请稍后重试!");
	}

	// 订单服务接口
	@RequestMapping("/orderInfo")
	public ResponseBase orderInfo() {
		System.out.println("orderInfo:" + "线程池名称:" + Thread.currentThread().getName());
		return setResultSuccess();
	}

	// Hystrix 有两种方式配置保护服务 通过注解和接口形式

}
```

reset方式的调用：

``` java
@Service
public class MemberService {
    @Autowired
    RestTemplate restTemplate;
    @HystrixCommand(fallbackMethod = "orderError")
    public List<String> getOrderByUserList() {
        return restTemplate.getForObject("http://service-member/getuser", List.class);
    }

    public List<String> orderError() {
        List<String> listUser = new ArrayList<String>();
        listUser.add("not orderUser list");
        return listUser;
    }
}
```

配置文件中添加：

``` yaml
###超时时间,不配置默认是一秒，实际开发之中需要进行配置
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 4000
```

主类中需要添加：

``` java
@EnableHystrix  // 开启断路器
```

feigin调用方式：

配置文件：

``` yaml
feign:
   hystrix:
     enabled: true # 开启断路器
#hystrix:
#   command: 
#     default: 
#       execution: 
#        isolation:
#         thread: 
#          timeoutInMilliseconds: 4000
```

service

``` java
@FeignClient(value = "service-member",fallback= MemberFeignService.class)
public interface MemberFeign {
	@RequestMapping("/getuser")
	public List<String> getOrderByUserList();
}
```

serviceImpl也就是服务降级调用的类

``` java
@Component
public class MemberFeignService implements MemberFeign {
    @Override
    public List<String> getOrderByUserList() {
            List<String> listUser = new ArrayList<String>();
            listUser.add("not orderUser list");
            return listUser;
    }
}
```

controller:

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

