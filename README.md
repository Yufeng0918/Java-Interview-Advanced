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


## 面试突击第二季

#### [面试分析](./docs/distributed-system/distributed-design.md)

#### [Dubbo和Spring Cloud分布式服务框架](./docs/distributed-system/core-architecture-principle%20.md)

#### [Dubbo和Spring Cloud框架原理](./docs/distributed-system/dubbo-framework-principle.md)

#### [服务注册中心](./docs/distributed-system/registration-center-guide.md)

#### [服务网关](./docs/distributed-system/gateway-model-selection.md)

#### [分布式系统生产实践](./docs/distributed-system/system-framework.md)

#### 分布式事务生产实践

#### 分布式锁生产实践



- [32、作业：独立画出自己系统的生产部署架构图，梳理系统和服务的QPS以及扩容方案](./docs/distributed-system/work-system-dilatation.md)
- [33、你们生产环境的服务是怎么配置超时和重试参数的？为什么要这样配置？](./docs/distributed-system/service-request-time-out.md)[代码下载点击这里哦!](https://github.com/shishan100/Java-Interview-Advanced/raw/master/docs/distributed-system/code/code4.zip)
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











