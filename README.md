# 互联网Java进阶面试训练营

## 面试突击第一季

#### 搜索引擎
- [lucene 和 es 的前世今生](/docs/high-concurrency/es-introduction.md)
- [es 的分布式架构原理能说一下么（es 是如何实现分布式的啊）？](/docs/high-concurrency/es-architecture.md)
- [es 写入数据的工作原理是什么啊？es 查询数据的工作原理是什么啊？底层的 lucene 介绍一下呗？倒排索引了解吗？](/docs/high-concurrency/es-write-query-search.md)
- [es 在数据量很大的情况下（数十亿级别）如何提高查询效率啊？](/docs/high-concurrency/es-optimizing-query-performance.md)
- [es 生产集群的部署架构是什么？每个索引的数据量大概有多少？每个索引大概有多少个分片？](/docs/high-concurrency/es-production-cluster.md)

#### [分布式消息队列](./docs/high-concurrency/mq-interview.md)

#### [分布式缓存](./docs/high-concurrency/why-cache.md)

#### [分库分表](./docs/high-concurrency/database-shard.md)
#### [分布式会话](./docs/distributed-system/distributed-session.md)
#### [分布式限流熔断降级](./docs/high-availability/hystrix-introduction.md)
- [Hystrix 介绍](./docs/high-availability/hystrix-introduction.md)
- [电商网站详情页系统架构](./docs/high-availability/e-commerce-website-detail-page-architecture.md)
- [Hystrix 线程池技术实现资源隔离](./docs/high-availability/hystrix-thread-pool-isolation.md)
- [Hystrix 信号量机制实现资源隔离](./docs/high-availability/hystrix-semphore-isolation.md)
- [Hystrix 隔离策略细粒度控制](./docs/high-availability/hystrix-execution-isolation.md)
- [深入 Hystrix 执行时内部原理](/docs/high-availability/hystrix-process.md)
- [基于 request cache 请求缓存技术优化批量商品数据查询接口](/docs/high-availability/hystrix-request-cache.md)
- [基于本地缓存的 fallback 降级机制](/docs/high-availability/hystrix-fallback.md)
- [深入 Hystrix 断路器执行原理](/docs/high-availability/hystrix-circuit-breaker.md)
- [深入 Hystrix 线程池隔离与接口限流](/docs/high-availability/hystrix-thread-pool-current-limiting.md)
- [基于 timeout 机制为服务接口调用超时提供安全保护](/docs/high-availability/hystrix-timeout.md)



## 互联网Java进阶面试训练营

### 第一季-分布式
- [01、互联网大厂面试要求：技术广度、技术深度、系统设计以及项目经验](/docs/distributed-system/distributed-design.md)
- [02、Java工程师面试突击第一季总结：你离一次成功的面试还差多少？](/docs/distributed-system/java-interview-season-1-summary.md)
- [03、《21天互联网Java进阶面试训练营》的课程说明](./docs/distributed-system/21-day-course-instructions.md)
- [04、作业：系统分析一下，自己距离大厂offer差在哪里？](/docs/distributed-system/homework.md)
- [05、感受一下BAT面试官对分布式技术的十几个面试连环炮!](/docs/distributed-system/BAT-interview-fire.md)
- [06、你们公司用的Dubbo？那你再额外说说Spring Cloud的核心架构原理？](/docs/distributed-system/core-architecture-principle%20.md)
- [07、基于Dubbo和Spring Cloud分别搭建一个电商系统来快速体验一下！](/docs/distributed-system/Dubbo-SpringCloud-experience.md)[代码下载点击这里哦!](https://github.com/shishan100/Java-Interview-Advanced/raw/master/docs/distributed-system/code/code.zip)
- [08、作业：你们的系统使用了哪种服务框架？为什么要这样技术选型？](/docs/distributed-system/distributed-framework-selection.md)
- [09、看过Dubbo源码吗？说说Dubbo的底层架构原理？](/docs/distributed-system/dubbo-framework-principle.md)
- [10、咱们来聊点深入的，说说Dubbo底层的网络通信机制原理！](/docs/distributed-system/dubbo-rock-bottom.md)
- [11、Dubbo框架从架构设计角度，是怎么保证极高的可扩展性的？](/docs/distributed-system/dubbo-augmentability.md)
- [12、作业：自己独立画出Dubbo的底层架构原理图](/docs/distributed-system/dubbo-independent-framework.md)
- [13、如果让你设计一个RPC框架，网络通信、代理机制、负载均衡等该如何设](/docs/distributed-system/rpc-design.md)
- [14、平时除了使用外，有研究过Spring Cloud的底层架构原理么？](/docs/distributed-system/springCloud-study-theory.md)
- [15、从底层实现原理的角度，对比一下Dubbo和Spring Cloud的优劣！](/docs/distributed-system/dubbo-vs-springCloud.md)
- [16、作业：自己独立画出Spring Cloud的架构原理图，RPC框架架构设计图！](/docs/distributed-system/springCloud-and-rpc-framework.md)
- [17、面试官：你们的服务注册中心进行过选型调研吗？对比一下各种服务注册中心！](/docs/distributed-system/registration-center-%20guide.md)
- [18、画图阐述一下你们的服务注册中心部署架构，生产环境下怎么保证高可用？](/docs/distributed-system/register-high-availability.md)
- [19、你们系统遇到过服务发现过慢的问题吗？怎么优化和解决的？](/docs/distributed-system/service-register-discovery.md)
- [20、作业：说一下自己公司的服务注册中心怎么技术选型的？生产环境中应该怎么优化？](/docs/distributed-system/register-production-optimize.md)
- [21、你们对网关的技术选型是怎么考虑的？能对比一下各种网关技术的优劣吗？](/docs/distributed-system/gateway-model-selection.md)
- [22、说说生产环境下，你们是怎么实现网关对服务的动态路由的？](/docs/distributed-system/dynamic-route.md)[代码下载点击这里哦!](https://github.com/shishan100/Java-Interview-Advanced/raw/master/docs/distributed-system/code/code2.zip)
- [23、如果网关需要抗每秒10万的高并发访问，你应该怎么对网关进行生产优化？](/docs/distributed-system/gateway-high-concurrency.md)
- [24、作业：你们公司的网关是怎么技术选型的，假设有高并发场景怎么优化？](/docs/distributed-system/gateway-technical.md)
- [25、如果需要部署上万服务实例，现有的服务注册中心能否抗住？如何优化？](/docs/distributed-system/registration-center-optimize.md)
- [26、你们是如何基于网关实现灰度发布的？说说你们的灰度发布方案？](/docs/distributed-system/gray-environment.md)[代码下载点击这里哦!](https://github.com/shishan100/Java-Interview-Advanced/raw/master/docs/distributed-system/code/code3.zip)
- [27、说说你们一个服务从开发到上线，服务注册、网关路由、服务调用的流程？](/docs/distributed-system/service-register-gateway-router.md)
- [28、作业：看看你们公司的服务注册中心能否支撑上万服务实例的大规模场景？](/docs/distributed-system/work-register.md)
- [29、画一下你们系统的整体架构图，说说各个服务在生产环境怎么部署的？](/docs/distributed-system/system-framework.md)
- [30、你们系统每天有多大访问量？每个服务高峰QPS多少？压测过服务最大QPS吗？](/docs/distributed-system/system-qps.md)
- [31、如果系统访问量比现在增加10倍，你们考虑过系统的扩容方案吗？](/docs/distributed-system/system-dilatation.md)
- [32、作业：独立画出自己系统的生产部署架构图，梳理系统和服务的QPS以及扩容方案](/docs/distributed-system/work-system-dilatation.md)
- [33、你们生产环境的服务是怎么配置超时和重试参数的？为什么要这样配置？](/docs/distributed-system/service-request-time-out.md)[代码下载点击这里哦!](https://github.com/shishan100/Java-Interview-Advanced/raw/master/docs/distributed-system/code/code4.zip)
- [34、如果出现服务请求重试，会不会出现类似重复下单的问题？](/docs/distributed-system/request-retry.md)
- [35、对于核心接口的防重幂等性，你们是怎么设计的？怎么防止重复下单问题？](/docs/distributed-system/interface-idempotence.md)
- [36、作业：看看自己系统的核心接口有没有设计幂等性方案？如果没有，应该怎么设计？](/docs/distributed-system/work-interface-idempotence.md)
- [37、画一下你们电商系统的核心交易链路图，说说分布式架构下存在什么问题？](/docs/distributed-system/deal-line.md)
- [38、针对电商核心交易链路，你们是怎么设计分布式事务技术方案的？](/docs/distributed-system/work-distributed-transaction.md)
- [39、对于TCC事务、最终一致性事务的技术选型，你们是怎么做的？如何调研的？](/docs/distributed-system/distributed-transaction-tcc.md)
- [40、作业：你们公司的核心链路是否有事务问题？分布式事务方案怎么调研选型？](/docs/distributed-system/work-distributed-transaction.md)
- [41、在搭建好的电商系统里，落地开发对交易链路的TCC分布式事务方案](/docs/distributed-system/tcc-landing-scheme.md)
- [42、你能说说一个TCC分布式事务框架的核心架构原理吗？](/docs/distributed-system/tcc-framework-principle.md)
- [43、现有的TCC事务方案的性能瓶颈在哪里？能支撑高并发交易场景吗？如何优化？](/docs/distributed-system/tcc-high-concurrence.md)
- [44、作业：如果对自己的系统核心链路落地TCC事务，应该如何落地实现？](/docs/distributed-system/work-tcc-landing-scheme.md)
- [45、你了解RocketMQ对分布式事务支持的底层实现原理吗？](/docs/distributed-system/rocketmq-transaction.md)
- [46、在搭建好的电商系统里，如何基于RocketMQ最终一致性事务进行落地开发？](/docs/distributed-system/rocketmq-eventual-consistency.md)
- [47、如果公司没有RocketMQ中间件，那你们如何实现最终一致性事务？](/docs/distributed-system/eventual-consistency.md)
- [48、作业：如果对自己的系统落地最终一致性事务，如何落地实现？](/docs/distributed-system/work-eventual-consistency.md)
- [49、你们生产系统中有哪个业务场景是需要用分布式锁的？为什么呢？](/docs/distributed-system/distributed-lock.md)
- [50、你们是用哪个开源框架实现的Redis分布式锁？能说说其核心原理么？](/docs/distributed-system/redis-distribute-lock.md)
- [51、如果Redis是集群部署的，那么集群故障时分布式锁还有效么？](/docs/distributed-system/hitch-redis-distribute-lock.md)
- [52、作业：自己梳理出来Redis分布式锁的生产问题解决方案](/docs/distributed-system/work-redis-distribute-lock.md)
- [53、如果要实现ZooKeeper分布式锁，一般用哪个开源框架？核心原理是什么？](/docs/distributed-system/zookeeper-distribute-lock.md)
- [54、对于ZooKeeper的羊群效应，分布式锁实现应该如何优化？](/docs/distributed-system/zookeeper-distribute-lock-optimize.md)
- [55、如果遇到ZooKeeper脑裂问题，分布式锁应该如何保证健壮性？](/docs/distributed-system/zookeeper-distribute-lock-split-brain.md)
- [56、作业：自己梳理出来ZooKeeper分布式锁的生产问题解决方案](/docs/distributed-system/zookeeper-distribute-lock-scheme.md)
- [57、在搭建好的电商系统中，落地开发分布式锁保证库存数据准确的方案](/docs/distributed-system/floor-distribute-lock.md)
- [58、你们的分布式锁做过高并发优化吗？能抗下每秒上万并发吗？](/docs/distributed-system/highly-concurrent-distribute-lock.md)
- [59、淘宝和京东的库存是怎么实现的？能不能不用分布式锁实现高并发库存更新？](/docs/distributed-system/distributed-lock-taobao-and-jingdong.md)
- [60、作业：自己系统的分布式锁在高并发场景下应该如何优化？](/docs/distributed-system/highly-concurrent-majorization-distributed-lock.md)
- [61、互联网Java工程师面试突击前两季总结以及下一季的规划展望](/docs/distributed-system/java-internet-interview-outlook.md)



### 第二季-高并发
### 第三季-微服务
### 第四季-海量数据
### 第五季-高性能
### 第六季-高可用











