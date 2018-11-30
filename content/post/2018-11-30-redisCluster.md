---
author: "皮智文"
date: 2018-11-30
title: "redis集群学习"
url: "/redis/cluster"
tags: ["redis","cluster"]
---

# redis集群部署运行

以window为例子

## 首先修改配置文件

redis-cluster.conf
~~~
1、配置文件需要配置持久化类型，这里选择aof
appendonly yes // 开启aof模式
appendfilename "appendonly.7001.aof" // aof名字
2、集群启动配置
cluster-enabled yes // 允许集群
cluster-config-file nodes-7001.conf // 集群配置文件，运行后自动生成
cluster-node-timeout 15000 // 集群节点响应超时
~~~

## 运行实例

进入redis目录下，运行redis-server.exe ./{"配置文件路径"}/name.conf
需要几个节点，就创建几个redis相关的配置文件，一般来说至少需要要3个master节点，每个主节点需要至少一个slave节点，因此至少需要6个节点
运行成功后只能算是开启了6个redis服务端，还不是集群

## 关联服务

6个服务端实例有了，就需要关联起来，通过使用 Redis 集群命令行工具 redis-trib 可以进行便携操作。
trib在redis源码中[trib地址](https://github.com/antirez/redis/tree/unstable/src),将redis-trib.rb文件放到redis安装目录下。
trib是rb（rubby）类型文件，所以需要安装rubby运行环境,[rubby安装地址](https://rubyinstaller.org/downloads/)。
在rubby目录下，安装redis驱动：gem install redis
redis目录下运行:./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005。

此时如果出现如下响应，表示集群关联成功。
Can I set the above configuration? (type 'yes' to accept): yes（这里输入yes表示同意设置）
~~~
C:\Program Files\Redis>redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: 7758fea3c85968a10453644d3b30479e8cf30485 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 25cd072ac06d892dd5cdd1260cb1d37cbac2ed0b 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 460ead6974a24a89b85755e44d4f2f4e6f260446 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 1f8196c9b4522abce8655f9cae1a1b640743d96d 127.0.0.1:7003
   replicates 7758fea3c85968a10453644d3b30479e8cf30485
S: 04fee0896a8ab3b7a43babb9010576ae9e7b2b0f 127.0.0.1:7004
   replicates 25cd072ac06d892dd5cdd1260cb1d37cbac2ed0b
S: 18b75190044f62a1dc214dee5396c0d4570ed297 127.0.0.1:7005
   replicates 460ead6974a24a89b85755e44d4f2f4e6f260446
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 7758fea3c85968a10453644d3b30479e8cf30485 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 460ead6974a24a89b85755e44d4f2f4e6f260446 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 25cd072ac06d892dd5cdd1260cb1d37cbac2ed0b 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 18b75190044f62a1dc214dee5396c0d4570ed297 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 460ead6974a24a89b85755e44d4f2f4e6f260446
S: 04fee0896a8ab3b7a43babb9010576ae9e7b2b0f 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 25cd072ac06d892dd5cdd1260cb1d37cbac2ed0b
S: 1f8196c9b4522abce8655f9cae1a1b640743d96d 127.0.0.1:7003
   slots: (0 slots) slave
   replicates 7758fea3c85968a10453644d3b30479e8cf30485
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
~~~

## 使用心得

redis服务端和集群启动有很多不一样，当对于客户端的使用却没有什么任何不同，当连接到集群中的任何一个节点，输入的信息都会同步到其他节点，同理用不同集群的节点客户端执行cmd命令都可以访问到相同的信息。

## go-redis集群模式原理

至少需要配置一个redis节点连接地址
~~~
redis.ClusterOptions{
    Addrs: []string{"127.0.0.1:7003"},
}
~~~

首先go-redis会在newClusterClient中通过7003这个节点客户端执行命令cluster slots获取其他所有节点，并且保存到ClusterClient实例中。

接下来可以通过
