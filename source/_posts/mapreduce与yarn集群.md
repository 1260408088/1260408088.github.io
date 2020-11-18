---
title: mapreduce与yarn集群
date: 2020-07-12 16:21:34
categories: [大数据]
tags: [大数据,linux]
---

​		统计HDFS的/wordcount/input/a.txt文件中的每个单词出现的次数——wordcount。明白了一点：可以在任何地方运行程序，访问HDFS上的文件并进行统计运算，并且可以把统计的结果写回HDFS的结果文件中；

但是，进一步思考：如果文件又多又大，用上面那个程序有什么弊端？

<!--more-->

<h1>慢！因为只有一台机器在进行运算处理！</h1>

如何变得更快？

核心思想：让我们的运算程序并行在多台机器上执行！

这就引出要说的重点.....

mapreduce程序应该是在很多机器上并行启动，而且先执行map task，当众多的maptask都处理完自己的数据后，还需要启动众多的reduce task，这个过程如果用用户自己手动调度不太现实，需要一个自动化的调度平台——hadoop中就为运行mapreduce之类的分布式运算程序开发了一个自动化调度平台——YARN

## yarn集群的配置

yarn集群中有两个角色：

主节点：Resource Manager 1台

从节点：Node Manager  N台

Resource Manager一般安装在一台专门的机器上

Node Manager应该与HDFS中的data node重叠在一起

需要修改**yarn-site.xml**配置文件，hadoop安装目录下的etc下

``` xml
配置文件的修改
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hdp-01</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

``` xml
yarn集群的内存与cpu的修改
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>2048</value>
</property>
<property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>2</value>
</property>

```

然后将配置文件复制到其他的几个机器上，使用scp命令

``` shell
scp yarn-site.xml hdf-02:$PWD // 复制文件到hdf-02的当前目录
```

然后就可以批量的启动yarn的集群，命令如下

``` shell
启动：
sbin/start-yarn.sh
停止：
sbin/stop-yarn.sh
```

需要注意，要在hdf-01上启动。

