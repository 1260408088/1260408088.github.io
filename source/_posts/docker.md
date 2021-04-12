---
title: docker
date: 2019-10-09 15:10:37
categories: 容器
tags: [docker,容器]
---

# docker 梳理

## docker的镜像的获取

​		获得docker的redis镜像

```powershell
docker search redis 
```

会得到关于redis的一些镜像

```po
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that…   7381                [OK]                
bitnami/redis                    Bitnami Redis Docker Image                      128                                     [OK]
sameersbn/redis                                                                  77                                      [OK]
grokzen/redis-cluster            Redis cluster 3.0, 3.2, 4.0 & 5.0               57                                      
rediscommander/redis-commander   Alpine image for redis-commander - Redis man…   31                                      [OK]
kubeguide/redis-master           redis-master with "Hello World!"                30                                      
redislabs/redis                  Clustered in-memory database engine compatib…   23                                      
oliver006/redis_exporter          Prometheus Exporter for Redis Metrics. Supp…   18                                      
arm32v7/redis                    Redis is an open source key-value store that…   17                                      
redislabs/redisearch             Redis With the RedisSearch module pre-loaded…   17                                      
webhippie/redis                  Docker images for Redis                         10                                      [OK]
s7anley/redis-sentinel-docker    Redis Sentinel                                  9                                       [OK]
insready/redis-stat              Docker image for the real-time Redis monitor…   8                                       [OK]
redislabs/redisgraph             A graph database module for Redis               8                                       [OK]
bitnami/redis-sentinel           Bitnami Docker Image for Redis Sentinel         7                                       [OK]

```

将name为redis的镜像拉下来

```po
docker pull redis 
```

查看本地的镜像列表

```po
docker images
```

```pow
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              01a52b3b5cd1        10 days ago         98.2MB
```

有了本地镜像就需要启动一下了

```po
 docker run -d  redis
```

74beaff66a23a436e49d5b1d65e7edb8a6fe3d01b5489b2a05d77b74b9ec1981

此种方式启动是后台启动。

执行：

```po
docker ps
```

```po
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
74beaff66a23        redis               "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        6379/tcp            trusting_dirac

```

显示后台中运行着的docker容器。

```po
docker stop 74be

```

参数解析：

| 参数名称     | 参数意义                                             |
| ------------ | ---------------------------------------------------- |
| IMAGE        | 创建容器时所使用的镜像<br/>                          |
| COMMAND      | 运行容器中的软件执行的命令<br/>                      |
| CREATED      | 容器的创建时间<br/>                                  |
| STATUS       | 容器的状态: UP 表示运行状态 Exited 表示关闭状态<br/> |
| PORTS        | 宿主机端口和容器中软件的端口的对应关系<br/>          |
| NAMES        | 容器的名称                                           |
| CONTAINER ID | 容器id<br/>                                          |

关闭CONTAINER ID  以74be开头的docker容器。

再次执行docker ps后无在运行中的容器。

容器停止后无法使用**docker ps**来查看，但是

```po
docker ps -a 

```

可以查看所有的容器（运行中的或者已经停止的）

```po
 docker ps -l

```

可以查看最后一次运行的容器。

```pow
docker ps -f status=exited 

```

可以查看停止的容器。镜像的可以删除的，同样已经创建的容器也是可以删除的

```po
docker rm $CONTAINER_ID/NAME  删除单个容器
docker rm `docker ps -a -q` 删除所有的容器

```

当然，此处的删除是退出状态下的容器。

执行删除后，容器就不存在了。可以用 **docker ps -f status=exited **验证一下

容器退出后还可以再次启动的

使用查看停止的容器命令

```po
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS               NAMES
74beaff66a23        redis               "docker-entrypoint.s…"   20 minutes ago      Exited (0) 10 minutes ago                          trusting_dirac
8e0c1259e6a8        redis               "docker-entrypoint.s…"   22 minutes ago      Exited (0) 21 minutes ago                          pensive_torvalds
d85bdd153358        redis               "docker-entrypoint.s…"   About an hour ago   Exited (0) 24 minutes ago                          happy_kowalevski
0a3afac8e147        redis               "docker-entrypoint.s…"   About an hour ago   Exited (0) About an hour ago                       stupefied_hypatia
23dfcd1a6787        redis               "docker-entrypoint.s…"   About an hour ago   Exited (0) About an hour ago                       romantic_golick

```

就CONTAINER ID讲容器启动

```po
docker start $CONTAINER_NAME/ID 

```

就上面的后台启动，可以这样进入容器内部

```po
docker exec -it redis /bin/bash

```

退出直接 **exit** 即可

初次启动参数：

| 参数名称 | 参数意义                                                     |
| -------- | ------------------------------------------------------------ |
| -i       | 运行容器                                                     |
| -t       | 表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端 |
| -d       | 在 run 后面加上-d 参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t 两个参数，创建后就会自动进去容器） |
| -name    | –name 为创建的容器命名                                       |
| -v       | 表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v 做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上 |
| -p       | 表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个－p 做多个端口映射 |

参数运用例子：

1. 创建交互形式的容器

```powershell
docker run -it --name=mjw_tomcat -p 8888:8080 -v /usr/local/webapps/:/usr/local/tomcat/webapps tomcat

```

2. 创建守护式的容器

```po
docker run -di --name=mjw_tomcat2 -p 8088:8080 -v /usr/local/webapps/:/usr/local/tomcat/webapps tomcat

```

另一种启动方式，直接交互式启动。

```po
sudo docker run  -it redis 

```

```po
1:C 06 Oct 2019 16:26:26.073 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 06 Oct 2019 16:26:26.073 # Redis version=5.0.6, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 06 Oct 2019 16:26:26.073 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

1:M 06 Oct 2019 16:26:26.074 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 06 Oct 2019 16:26:26.074 # Server initialized
1:M 06 Oct 2019 16:26:26.075 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 06 Oct 2019 16:26:26.075 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 06 Oct 2019 16:26:26.075 * Ready to accept connections


```

但是你**Ctrl+c**退出后容器也就直接关闭了。(这种方式应该不推荐使用)

## docker的容器的备份

将容器保存为镜像

```properties
docker commit 容器名称 保存的新镜像的名

```

讲镜像打包

```po
docker save –o 打包以后的文件名称 镜像名称

```

打包后的镜像的回复

```po
docker load -i 镜像保存的tar包 

```

https://blog.csdn.net/Min_JW/article/details/83686051 看这篇文章，对两种启动方式有详细的说明，端口的映射与磁盘的映射