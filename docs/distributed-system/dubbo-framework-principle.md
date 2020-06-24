# Dubbon和Spring Cloud原理

聊**分布式**这块，**Dubbo相关的原理**，**Spring Cloud相关的原理**，有的面试官可能会这样问，你有没有看过**Dubbo或者Spring Cloud的源码呢**？**技术广度**、**技术深度**、**项目经验**、**系统设计**、**基本功**



## Dubbon 原理

分布式系统

拆分为了多个子系统之后，各个系统之间如何通过Spring Cloud服务框架来进行调用，Dubbo框架来进行调用

![Dubbo核心架构原理](../distributed-system/images/dubbo-framework-principle.png)

### 服务注册中心

+ 为服务提供者注册
+ 为服务消费者提供服务列表



### 消费者

+ **动态代理**：Proxy，代理远程接口
+ **负载均衡**：Cluster，负载均衡，故障转移
  + 获取服务列表
  + LoadBalancer选择主机
+ **通信协议**：Protocol，filter机制，http、rmi、dubbo等协议, 比如想要调用的是，DemoService里的sayHello接口
  + http: /demoService/sayHello?name=leo
  + dubbo: nterface=demoService|method=sayHello|params=name:leo

+ **信息交换**：Exchange对于你的协议的格式组织好的请求数据，需要进行一个封装，Request
+ **网络通信**：Transport，netty、mina
+ **序列化**：封装好的请求如何序列化成二进制数组，通过netty/mina发送出去





### 提供者

+ **网络通信**：Transport，基于netty/mina实现的Server
+ **信息交换**：Exchange 解析 Response
+ **通信协议**：通过预定的Protocol进行解析，filter机制
+ **动态代理**：调用服务端的Proxy, 动态代理再去找具体的实现



![Dubbo底层通信原理](../distributed-system/images/dubbo-rock-bottom.png)
网络通信这块用netty来举例，NIO来实现的，一台机器同时抗高并发的请求

+ 一个acceptor负责接收请求的selector， serverSocketChannel注册一个accept事件在selector上
+ 有几个线程，每个线程负责一个selector，一个selector对应多个socketChannel的read/write事件



![Dubbo底层通信原理](../distributed-system/images/dubbo-rock-bottom.png)



### 可扩展性

+ 是核心的组件全部接口化，组件和组件之间的调用，必须全部是依托于接口，去动态找配置的实现类，如果没有配置就用他自己默认的
+ 提供一种自己实现的组件的配置的方式，比如说你要是自己实现了某个组件，配置一下，人家到时候运行的时候直接找你配置的那个组件即可，作为实现类，不用自己默认的组件了



### RPC 框架

从系统设计的角度，来考虑一下，到底如果要设计一个RPC框架。

**动态代理**：比如消费者和提供者，其实都是需要一个实现某个接口的动态代理的，RPC框架的一切的逻辑细节，都是在这个动态代理中实现的，动态代理里面的代码逻辑就是你的RPC框架核心的逻辑。 JDK提供了API，去创建针对某个接口的动态代理

**服务注册和拉取**

自己手撸一个，服务去注册，其他服务去拉取注册表进行发现

+ ZooKeeper**，稍微自己上网百度搜索一下，**ZooKeeper**入门使用教程，基本概念和原理，还有基本的使用，了解一下**

+ Cluster层，从本地缓存的服务注册表里获取到要调用的服务的机器列表

**负载均衡**: 从服务的机器列表中采用负载均衡算法从里面选择出来一台机器

**协议**，**序列化机制**，**底层网络通信框架**，比如**netty，mina**现在用的比较少，

+ 序列化一个复杂的请求数据序列化成二进制的字节数组
+ 反序列化就是从字节数组变成请求数据结构



## Spring Cloud 原理

**Spring Cloud的源码**，外发布一个接口，实际上就是支持**http协议**的，对外发布的就是一个最最普通的**Spring MVC的http接口**

![Eureka服务注册中心的原理](../distributed-system/images/springCloud-study-theory.png)

### Eureka

**服务注册**，服务注册用了两级缓存，减少读写的冲突。服务注册表后，修改ReadWrite缓存，ReadOnly缓存定时周期30s拉取ReadWrite缓存配置

**心跳**, 注册服务每隔30秒发送心跳给注册中心



### Feign

对一个接口打了一个注解，一定会针对这个注解标注的接口生成**动态代理**，然后对feign的动态代理去调用他的方法的时候，此时会在底层生成http协议格式的请求，/order/create?productId=1



### Ribbon

使用HTTP通信的框架组件，**HttpClient** 先为Ribbon去从本地的Eureka注册表的缓存里获取出来对方机器的列表，然后进行负载均衡，选择一台机器出来，接着针对那台机器发送Http请求过去即可



### Zuul

配置一下不同的请求路径和服务的对应关系，请求到了网关，直接查找到匹配的服务，然后就直接把请求转发给那个服务的某台机器，**Ribbon从Eureka本地的缓存列表里获取一台机器，负载均衡，把请求直接用HTTP通信框架发送到指定机器上去**





## Dubbon 和 Spring Cloud 比较

### 交互协议比较

+ **Dubbo，RPC的性能比HTTP的性能更好，并发能力更强，经过深度优化的RPC服务框架，性能和并发能力是更好一些**
+ **Spring Cloud这套架构原理，走HTTP接口和HTTP请求，就足够满足性能和并发的需要了，没必要使用高度优化的RPC服务框架**

+ 对很多中小型公司而言，性能、并发，并不是最主要的因素

### 框架定位

+ Dubbo之前的一个定位，就是一个单纯的服务框架而已，不提供任何其他的功能，配合的网关还得选择其他的一些技术

+ **Spring Cloud**，中小型公司用的特别多，老系统从**Dubbo迁移到Spring Cloud**，新系统都是用**Spring Cloud来进行开发，全家桶，主打的是微服务架构里，组件齐全，功能齐全。网关直接提供了，分布式配置中心，授权认证，服务调用链路追踪，Hystrix可以做服务的资源隔离、熔断降级、服务请求QPS监控、契约测试、消息中间件封装、zookeeper封装**

**Spring Cloud**原来支持的一些技术慢慢的未来会演变为，跟阿里技术体系进行融合，**Spring Cloud Alibaba**，阿里技术会融入**Spring Cloud**里面去