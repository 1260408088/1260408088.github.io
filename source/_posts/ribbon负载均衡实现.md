---
title: ribbon负载均衡实现
date: 2019-05-24 12:15:37
categories: springcloud
tags: rpc
---

# 1.负载均衡原理（轮训）

客户端请求的总次数对服务器集群列表数目进行求余运算

<!-- more-->

- 当前请求总数为1，服务器列表的集群为2，1%2=1。所以当前访问下标为 List [1] 的服务器
- 当前请求总数为2，服务器列表的集群为2，2%2=0。所以当前访问下标为 List [0] 的服务器
- 当前请求总数为3，服务器列表的集群为2，2%2=0。所以当前访问下标为 List [1] 的服务器

说明总数是从最开始访问开始累加的。

# 2.带权重负载均衡解析

轮训的方式在实际开发之中并不实际，因为不可能每一台服务器的性能都是相同的，所以带权重的更为实际一些

求得最大公约数与最大权重值，每满足一次返回服务器后，减去最小权重值然后接着循环！直到将服务器列表循环完毕一次。

``` java
public class WeightRoundRobin {
	
    /**上次选择的服务器*/  
    private int currentIndex = -1;  
    /**当前调度的权值*/  
    private int currentWeight = 0;  
    /**最大权重*/  
    private int maxWeight;  
    /**权重的最大公约数*/  
    private int gcdWeight;  
    /**服务器数*/  
    private int serverCount;  
    private List<Server> servers = new ArrayList<Server>(); 
    
    /*
     * 得到两值的最大公约数
     */
    public int greaterCommonDivisor(int a, int b){  
    	if(a % b == 0){
			return b;
		}else{
			return greaterCommonDivisor(b,a % b);
		}
    }
    /*
     * 得到list中所有权重的最大公约数，实际上是两两取最大公约数d，然后得到的d
     * 与下一个权重取最大公约数，直至遍历完
     */
    public int greatestCommonDivisor(List<Server> servers){  
        int divisor = 0;  
        for(int index = 0, len = servers.size(); index < len - 1; index++){  
            if(index ==0){  
                divisor = greaterCommonDivisor(  
                            servers.get(index).getWeight(), servers.get(index + 1).getWeight());  
            }else{  
                divisor = greaterCommonDivisor(divisor, servers.get(index).getWeight());  
            }  
        }  
        return divisor;  
    }
    
    /*
     * 得到list中的最大的权重
     */
    public int greatestWeight(List<Server> servers){  
        int weight = 0;  
        for(Server server : servers){  
            if(weight < server.getWeight()){  
                weight = server.getWeight();  
            }  
        }  
        return weight;  
    }
    
    /** 
     *  算法流程：  
     *  假设有一组服务器 S = {S0, S1, …, Sn-1} 
     *  有相应的权重，变量currentIndex表示上次选择的服务器 
     *  权值currentWeight初始化为0，currentIndex初始化为-1 ，当第一次的时候返回 权值取最大的那个服务器， 
     *  通过权重的不断递减 寻找 适合的服务器返回
     */  
     public Server getServer(){  
         while(true){  
             currentIndex = (currentIndex + 1) % serverCount;  
             if(currentIndex == 0){  
                 currentWeight = currentWeight - gcdWeight;  
                 if(currentWeight <= 0){  
                     currentWeight = maxWeight;  
                     if(currentWeight == 0){  
                         return null;  
                     }  
                 }  
             }  
             if(servers.get(currentIndex).getWeight() >= currentWeight){  
                 return servers.get(currentIndex);  
             }  
         }  
     } 
     
     public void init(){  
         servers.add(new Server("192.168.191.1", 1));  
         servers.add(new Server("192.168.191.2", 2));  
         servers.add(new Server("192.168.191.4", 4));  
         servers.add(new Server("192.168.191.8", 8));  
        
           
         maxWeight = greatestWeight(servers);  
         gcdWeight = greatestCommonDivisor(servers);  
         serverCount = servers.size();  
     }  
     
     public static void main(String args[]){
    	  WeightRoundRobin weightRoundRobin = new WeightRoundRobin();  
          weightRoundRobin.init();  
            
          for(int i = 0; i < 15; i++){  
              Server server = weightRoundRobin.getServer();  
              System.out.println("server " + server.getIp() + " weight=" + server.getWeight());  
          }  
     }
 
}
 
class Server{  
	 private String ip;  
     private int weight;  
     
     public Server(String ip, int weight) {  
         this.ip = ip;  
         this.weight = weight;  
     }
 
	public String getIp() {
		return ip;
	}
 
	public void setIp(String ip) {
		this.ip = ip;
	}
 
	public int getWeight() {
		return weight;
	}
 
	public void setWeight(int weight) {
		this.weight = weight;
	}  
     
}
```



# 3.手写ribbon的负载均衡

在服务的消费端写一个controller,原理就是轮训调用(使用了DiscoveryClient)

``` java
@RestController
public class ExtController {
    @Autowired
    DiscoveryClient discoveryClient;
    @Autowired
    RestTemplate restTemplate;
    private int count = 1;

    @RequestMapping("getblance")
    public String getBance() {
        ServiceInstance instance = getInstance();
        String url = instance.getUri().toString() + "getuser";
        String result = restTemplate.getForObject(url, String.class);
        return result;
    }

    public ServiceInstance getInstance() {
        List<ServiceInstance> instances = discoveryClient.getInstances("service-member");
        if (instances == null || instances.size() <= 0) {
            return null;
        }
        int totalsize = instances.size();
        int serverindex = count % totalsize;
        count++;
        ServiceInstance serviceInstance = instances.get(serverindex);
        return serviceInstance;
    }
}
```

写这个例子的时候，需要讲本地在配置的 @LoadBalanced 给注释掉