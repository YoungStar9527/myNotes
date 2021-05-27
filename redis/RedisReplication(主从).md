# 1 redis瓶颈/如何支撑高并发

## 1.1 redis高并发跟整个系统的高并发之间的关系

redis是高并发的重要支撑，不可避免，要吧底层的缓存搞的很好

mysql 高并发，做到的话，需要经过一系列的分库分表等复杂操作，一般是对事务性有比较高要求的，比如订单系统等，才会才有mysql做高并发,mysql的并非QPS到几万就算比较高了

真正的超高并非,QPS上十万，甚至上百万，比如电商的商品详情页等，光是redis是不够的，但是redis是整个大型缓存架构中，支撑高并发架构里面，非常重要的一个环节

首先，底层的缓存中间件。缓存系统，必须能够支撑的起我们说的那种高并发，其次。再经过良好的整体的银存架构的设计多级缓存架构、热点缓存，支撑真正的上十万，甚至上百万的高并发。

**PS:总结一句话，redis是高并发的重要支撑，在真正上百万的高并发系统需要相关的复杂设计，而redis是其中的一个重要环节**

## 1.2 redis不能支撑高并发的瓶颈在哪里

单机

![image-20210509205138363](RedisReplication(主从).assets/image-20210509205138363.png)

## 1.3 如果redis要支持超过10万+的并发，那应该怎么做

​	单机的redis几乎不太可能说oPs超过10万+，除非一些特殊情况，比如你的机器性能特别好，配置特别高，物理机，维护做的特别好，而且你的整体的操作不是太复杂单机在几万
读写分离，一般来说，对缓存，一般都是用来支撑读高并发的，写的请求是比较少的，可能写请求也就一秒钟几千，一两千大量的请求都是读，一秒钟二十万次读
读写分离

![image-20210509205519182](RedisReplication(主从).assets/image-20210509205519182.png)

主从架构 ->读写分离->支撑10万+读QPS的架构

# 2 redis replication(主从)

redis replication

redis主从架构 ->读写分离架构 ->可支持水平扩展的读高并发架构

## 2.1 redis replication的核心机制

(1) redis采用异步方式复制数据到slave节点，不过redis 2.8开始，slave node会周期性地确认自己每次复制的数据量

(2)一个master node是可以配置多个slave node的

(3) slave node也可以连接其他的slave node

(4) slave node做复制的时候，是不会block master node的正常工作的

(5) slave node在做复制的时候，也不会block对自己的查询操作，它会用旧的数据集来提供服务,但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了

(6) slave node主要用来进行横向扩容，做读写分离，扩容的slave node可以提高读的吞吐量

**PS:slave节点和高可用有很大关系**

**PS:新增大量的读请求的话，可以采取新增slave node节点作为读的节点 方案**

## 2.2、master持久化对于主从架构的安全保障的意义

​	如果采用了主从架构，那么建议**必须开启master node的持久化**!

​	不建议用slave node作为master node的数据热备，因为那样的话，如果你关掉master的持久化，可能在master宕机重启的时候数据是空的，然后可能一经过复制，salve node数据也丢了

​	master ->RDB和AOF都关闭了→>全部在内存中

​	master宕机，重启，是没有本地数据可以恢复的，然后就会直接认为自己IDE数据是空的master就会将空的数据集同步到slave上去，所有slave的数据全部清空 100%的数据丢失master节点，必须要使用持久化机制

​	第二个，master的各种备份方案，要不要做，万一说本地的所有文件丢失了﹔从备份中挑选一份RDB去恢复master 这样才能确保master启动的时候，是有数据的

​	即使采用了后续讲解的高可用机制，slave node可以自动接管master node，但是也可能sentinel还没有检测到master failure， master mode就自动重启了，还是可能导致上面所有slave node数据清空故障

**PS:总结一句话就是：master必须开启持久化，不然可能会导致master及其所有slave所有数据全部丢失**

## 2.3 主从复制原理

### 2.3.1、主从架构的核心原理

​	当启动一个slave node的时候，它会发送一个PSYNC命令给master node

​	如果达是slave move重新连接master move，那么master move仅仅会复制给slave部分缺少的数据，否则如果是slave node第一次连接master node，那么会触发一次full resynchronization

​	开始full resynchronization的时候，master会启动一个后台线程，开始生成一份RDB快照文件，同时还会将从客户增收到的所有写全会缓存在内存中。RDB文件生成完毕之后， master 会将这个RDB发送给slave， slave会先写入本地磁盘，然后再从本地磁盘加载到内存中。然后master会将内存中缓存的写命令发送给slave， slave也会同步这些数据。

​	slave node如果跟master mode有网络故障，断开了连接，会自动重连。master 如果发现有多个slave node都来重新连接，仅仅会启动一个RDB save操作，用一份数据服务所有slave node。

![image-20210510062849218](RedisReplication(主从).assets/image-20210510062849218.png)



### 2.3.2 主从复制的断点续传


​	从redis 2.8开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去,而不是从头开始复制一份

​	master node会在内存中创建一个backlog，master和slave都会保存一个replica offset还有一个master id，offset就是保存在backlog中的。如果master和slave网络连接断掉了，slave会让master从上次的replica offset开始继续复制

​	但是如果没有找到对应的offset，那么就会执行一次resynchronization

### 2.3.3 无磁盘化复制

​	master在内存中直接创建rdb，然后发送给slave，不会在自己本地落地磁盘了

```shell
repl-diskless-sync no
#是否开启无磁盘化复制,no 关闭 yes开启
repl-diskless-sync-delay 5
#等待一定时长再开始复制，因为要等更多slave重新连接过来

```

### 2.3.4 过期key处理

​	slave不会过期key，只会等待master过期key，如果master过期了一个key，或者通过LRU淘汰了一个key，那么会模拟一条del命令发送给slave

## 2.4 深入剖析redis replication原理和完整运行流程

### 2.4.1 复制的完整流程

​	(1) slave node启动，仅仅保存master node的信息，包括master node的host和ip，但是复制流程没开始master host和ip是从哪儿来的，redis.conf里面的slaveof配置的

​	(2) slave node内部有个定时任务，每秒检查是否有新的master node要连接和复制，如果发现，就跟master node建立socket网络连接

​	(3) slave node发送ping命令给master node

​	(4)口令认证，如果master设置了requirepass，那么salve node必须发送masterauth的口令过去进行认证

​	(5) master node第一次执行全量复制，将所有数据发给slave node

​	(6) master node后续持续将写命令，异步复制给slave node

![image-20210510074749850](RedisReplication(主从).assets/image-20210510074749850.png)

### 2.4.2 数据同步相关的核心机制

​	指的就是第一次slave连接master的时候，执行的全量复制，那个过程里面你的一些细节的机制

​	(1) master和slave都会维护一个offset slave每秒都会上报自己的offset给master，同时master也会保存每个slave的offset
这个倒不是说特定就用在全量复制的，主要是master和slave都要知道各自的数据的offset，才能知道互相之间的数据不一致的情况

​	(2) backlog master node有一个backlog，驮认是1MB大小 master node给slave node复制数据时，也会将数据在backlog中同步写一份backlog主要是用来做全量复制中断候的增量复制的

​	(3) master run id

​	 info server，可以看到master run id 如果根据host+ip定位master node，是不靠谱的，如果master node重启或者数据出现了变化，那么slave node应该根据不同的run id区分，run id不同就做全量复制。

​	 如果需要不更改run id重启redis，可以使用redis-cli debug reload命令

​	(4) psync

​	从节点使用psync从master node进行复制，psync runid offset 

​	master node会根据自身的情况返回响应信息，可能是FULLRESYNC runid offset触发全量复制，可能是CONTINUE触发增量复制

### 2.4.3 全量复制

​	(1) master执行bgsave，在本地生成一份rdb快照文件

​	(2) master node将rdb快照文件发送给salve node，如果rdb复制时间超过6o秒(repl-timeout)，那么slave node就会认为复制失败，可以适当调节大这个参数

​	(3）对于千兆网卡的机器，一般每秒传输100MB，6G文件，很可能超过60s

​	(4) master node在生成rdb时，会将所有新的写命令缓存在内存中，在slave node保存了rdb之后，再将新的写命令复制给slave node

​	(5)client-output-buffer-limit slave 256MB 64MB 60，如果在复制期间，内存缓冲区持续消耗超过64MB，或者一次性超过256MB ，那么停止复制，复制失败

​	(6) slave node接收到rdb之后，清空自己的旧数据，然后重新加载rdb到自己的内存中，同时基于旧的数据版本对外提供服务

​	(7)如果slave node开启了AOF，那么会立即执行BGREMRITEAOF，重写AOF

​		rdb生成、rdb通过网络拷贝、slave旧数据的清理、slave aof rewrite，很耗费时间

  	如果复制的数据量在4G~6G之间，那么很可能全量复制时间消耗到1分半到2分钟

### 2.4.4 增量复制

​	(1)如果全量复制过程中，master-slave网络连接断掉，那么salve重新连接master时，会触发增量复制

​	(2) master直接从自己的backlog中获取部分丢失的数据，发送给slave node，默认backlog就是1MB

​	(3) master就是根据slave发送的psync中的offset来从backlog中获取数据的

### 2.4.5 heartbeat

​	主从节点互相都会发送heartbeat信息

​	master默认每隔10秒发送一次heartbeat,slave node每隔1秒发送一个heartbeat

### 2.4.6 异步复制

​	master每次接收到写命令之后，先在内部写入数据，然后异步发送给slave node

## 2.4 主从相关配置

### 2.4.1 主从配置

在从节点配置下列命令即可(slave/replica)

```shell
replicaof 127.0.0.1 6379
#reids 5.0之后replicaof <master-ip> <port>
slaveof 127.0.0.1 6379 
#reids 5.0之前replicaof <master-ip> <port>
replica-read-only yes
#从节点只读(强制读写分离)-默认开启 5.0之前为slave-read-only

```

**PS:目前5.0之后还兼容slave的配置，将replica改成slave即可，如replicaof改成slaveof在5.0之后也能使用**

### 2.4.2 主从密码连接

可选-集群安全认证(也就是密码)(生产环境一般需要设置)

主节点配置

```shell
requirepass 123456
#requirepass <password> 
#设置密码(启用安全认证)
```

从节点配置

```shell
masterauth 123456
#masterauth <password> 
#master连接口令
```

redis-cli中密码验证

```shell
auth redis-pass
#auth <password>
```



### 2.4.3 基本信息查看

```shell
info replication
#redis-cli中 使用该命令查看
```

主节点信息

![image-20210511211006691](RedisReplication(主从).assets/image-20210511211006691.png)

从节点信息

![image-20210511210952274](RedisReplication(主从).assets/image-20210511210952274.png)

未配置主从节点信息

![image-20210511210929090](RedisReplication(主从).assets/image-20210511210929090.png)

## 2.5 QPS压测

### 2.5.1 压测

在redis目录下的 redis-benchmark工具

```shell
./redis-benchmark -h 127.0.0.1
#redis-benchmark -h <host>
#运行此压测工具即可使用默认参数进行压测
#redis-benchmark -h 查看相关参数配置
```

常用参数使用

```shell
 -c <clients>       Number of parallel connections (default 50) 
 #默认50个客户端 可调整
 -n <requests>      Total number of requests (default 100000)
  #默认100000个请求 可调整
 -d <size>          Data size of SET/GET value in bytes (default 3)
  #默认每个key的值大小3b 可调整
 ......
```

实际效果

虚拟机 1内核 4核心

命令：

```shell
./redis-benchmark -h 192.168.31.103 -d 100

```

set命令

![image-20210512075904216](RedisReplication(主从).assets/image-20210512075904216.png)

get命令

![image-20210512075934520](RedisReplication(主从).assets/image-20210512075934520.png)

### 2.5.2 一些说明

​	1 大公司具体项目配置大部分是云平台虚拟机，配置较低，且业务较复杂，key值数据较大，可能1k及以上。比如4核4g内存在真实项目中，单机redis能做到几万就差不多了。

​	2 redis至少能提供上万的QPS，在测试1核心1线程,1g的虚拟机上，都能达到上万。如果服务器性能特别强,维护的很好，redis单机QPS也能达到几万,十几万，二十万不等

​	3 QPS的两大杀手：一个是复杂操作，lrange相等命令。第二个是打value，可能1k及以上的数据

​	4 水平扩容redis读节点，提升吞吐量，在其他服务器上搭建redis从节点，单个节点读请求QPS在5万左右，两个redis从节点，读请求打到两台机器上去，承载整个集群读QPS在10万+











