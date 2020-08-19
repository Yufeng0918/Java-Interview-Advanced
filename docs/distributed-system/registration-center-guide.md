# 服务注册中心

非常常见的一个技术面试题，但凡只要是聊到分布式这块，Dubbo，Spring Cloud，服务注册中心，你们当时是怎么选型和调研的，你们最终是选择了哪块技术呢？你选择这块技术的原因和理由是什么呢？

**Eureka vs ZooKeeper**

**Dubbo**作为服务框架的，一般注册中心会选择zk

**Spring Cloud**作为服务框架的，**一般服务注册中心会选择Eureka**

**Consul, Nacos，**普及型还没那么广泛，我会在面试训练营课程里增加对应的内容，给大家去进行补充



## Eurkea vs Zookeeper

### 服务注册发现的原理

**集群模式**, Eureka，peer-to-peer，部署一个集群，但是集群里每个机器的地位是对等的，各个服务可以向任何一个Eureka实例服务注册和服务发现，集群里任何一个Euerka实例接收到写请求之后，会自动同步给其他所有的Eureka实例

![ZooKeeper](../distributed-system/images/eureka-register.png)



**主从模式**， ZooKeeper，服务注册和发现的原理，Leader + Follower两种角色，只有Leader可以负责写也就是服务注册，他可以把数据同步给Follower，读的时候leader/follower都可以读

![ZooKeeper](../distributed-system/images/zookeeper-register.png)



###  一致性保障：CP or AP

**CAP，C是一致性，A是可用性，P是分区容错性**，CP 和 AP两种选型

**CP**: **ZooKeeper**是有一个leader节点会接收数据， 然后同步写其他节点，一旦leader挂了，要重新选举leader，这个过程里为了保证C，就牺牲了A，不可用一段时间，但是一个leader选举好了，那么就可以继续写数据了，保证一致性

**AP**: **Eureka**是peer模式，可能还没同步数据过去，结果服务挂机，此时还是可以继续从别的机器上拉取注册表，但是看到的就不是最新的数据了，但是保证了可用性，强一致，**最终一致性**



### 服务注册发现的时效性

zookeeper，时效性更好，注册或者是挂了，一般秒级就能感知到

eureka，**默认配置非常糟糕**，服务发现感知要到几十秒，甚至分钟级别，上线一个新的服务实例，到其他人可以发现他，极端情况下，可能要1分钟的时间，ribbon去获取每个服务上缓存的eureka的注册表进行负载均衡

服务故障，隔60秒才去检查心跳，发现这个服务上一次心跳是在60秒之前，隔60秒去检查心跳，超过90秒没有心跳，才会认为服务挂了，2分钟都过去。 30秒，才会更新缓存，30秒，其他服务才会来拉取最新的注册表。三分钟都过去了，如果你的服务实例挂掉了，此时别人感知到，可能要两三分钟的时间，一两分钟的时间，很漫长



### 容量

zookeeper，不适合大规模的服务实例，**因为服务上下线的时候，需要瞬间推送数据通知到所有的其他服务实例**，所以一旦服务规模太大，到了几千个服务实例的时候，会导致网络带宽被大量占用。**同步的时间也会更久**。

eureka，也很难支撑大规模的服务实例，因为每个**eureka实例都要接受所有的请求**，实例多了压力太大，扛不住，也很难到几千服务实例

之前dubbo技术体系都是用zk当注册中心，spring cloud技术体系都是用eureka当注册中心这两种是运用最广泛的，但是现在很多中小型公司以spring cloud居多，所以后面基于eureka说一下服务注册中心的生产优化



### 高可用

Eureka 架构保证高可用，多机房, 多数据中心部署



## Eureka 配置参数优化

zookeeper服务注册和发现，都是很快的

**eureka必须优化参数**, **关闭自我保护机制**

```properties
//服务注册中心缓存更新时间
eureka.server.responseCacheUpdateIntervalMs = 3000
//服务端检查心跳
eureka.server.evictionIntervalTimerInMs = 6000
//服务过期时间
eureka.instance.leaseExpirationDurationInSeconds = 6

//服务消费者拉取注册表间隔时间
eureka.client.registryFetchIntervalSeconds = 3000
//消费端心跳时间
eureka.client.leaseRenewalIntervalInSeconds = 3
```

**服务发现的时效性变成秒级，几秒钟可以感知服务的上线和下线**



## 技术选型

**服务注册中心，eureka、zk、consul，原理画图画清楚**

**数据一致性，CP、AP**

服务注册、故障 和发现的时效性是多长时间？ 注册中心最大能支撑多少服务实例？

如何部署的，几台机器，每台机器的配置如何，会用比较高配置的机器来做，8核16G，16核32G的高配置机器来搞，基本上可以做到每台机器每秒钟的请求支撑几千绝对没问题

**服务注册、故障以及发现的时效性优化，用eureka的话，可以尝试一下，配合我们讲解的那些参数，优化一下时效性，服务上线，故障到发现是几秒钟的时效性**

**zk一旦服务挂掉，zk感知到以及通知其他服务的时效性，服务注册到zk之后通知到其他服务的时效性，leader挂掉之后可用性是否会出现短暂的问题，为了去换取一致性**



## 高并发网关分片

![分布式注册中心](/Users/daiyu/dev/idea/architect/Java-Interview-Advanced/docs/distributed-system/images/registration-center-optimize.png)

+ eureka：peer-to-peer，每台机器都是高并发请求，有瓶颈
+ zookeeper：服务上下线，全量通知其他服务，网络带宽被打满，有瓶颈

### 技术设计

+ 分布式服务注册中心
+ 分片存储服务注册表
+ 横向扩容
+ 每台机器均摊高并发请求
+ 各个服务主动拉取
+ 避免反向通知网