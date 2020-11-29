

# 分布式事务

分布式事务了解吗？你们是如何解决分布式事务问题的。只要聊到你做了分布式系统，必问分布式事务，你对分布式事务一无所知的话，确实会很坑，你起码得知道有哪些方案，一般怎么来做，每个方案的优缺点是什么。分布式系统成了标配，而分布式系统带来的**分布式事务**也成了标配了。因为你做系统肯定要用事务吧，如果是分布式系统，肯定要用分布式事务吧。先不说你搞过没有，起码你得明白有哪几种方案，每种方案可能有啥坑？比如 TCC 方案的网络问题、XA 方案的一致性问题。



## 0. 理论基础

#### CAP

+ consistence: 强一致性，数据一定要全部节点同步才可以对外服务。还有弱一致性，最终一致性
+ available：可用性，客户端对系统可读可写
+ tolerance-partition： 分区容忍性，分布式系统各个节点之间不能进行通信依然可以对外提供服务

CAP不可能完全拥有，因为是分布式系统，所以P一定是具备的。所以系统原则要么是AP，要么你是CP

#### BASE

+ Basic Avialable：基本可用， CAP可用同时实现，但是不要求100%实现。网络故障时，不同的节点都可以查询，有的节点可以访问数据库，有点节点无法访问数据库。这就是不一致状态。正常情况下，可以到每个节点查询，这时候可以强制都去主节点查询，而避免查询结果不一致。**需要对主节点进行限流和服务降级避免流量过载。**

+ Soft State：不一致的状态就是软状态

+ Eventual Consistency：最终一致性



## 1. 技术方案

分布式事务的实现主要有以下 5 种方案：

- 2PC 方案（同步方案）
- TCC 方案（同步方案）
- 本地消息表 （异步方案）
- 可靠消息最终一致性方案 （异步方案）
- 最大努力通知方案 （异步方案）

### 2-Pharse-Commit提交方案
所谓的 XA 方案，即：两阶段提交，有一个**事务管理器**的概念，负责协调多个数据库（资源管理器）的事务，事务管理器先问问各个数据库你准备好了吗？如果每个数据库都回复 ok，那么就正式提交事务，在各个数据库上执行操作；如果任何其中一个数据库回答不 ok，那么就回滚事务。

这种分布式事务方案，**比较适合单块应用里，跨多个库的分布式事务**，而且因为严重依赖于数据库层面来搞定复杂的事务，效率很低，绝对不适合高并发的场景。那么基于 `Spring + JTA` 来实现。

这个方案很少用，一般来说**某个系统内部如果出现跨多个库**的这么一个操作，是**不合规**的。我可以给大家介绍一下， 现在微服务，一个大的系统分成几十个甚至几百个服务。一般来说，我们的规定和规范，是要求**每个服务只能操作自己对应的一个数据库**。

如果你要操作别的服务对应的库，不允许直连别的服务的库，违反微服务架构的规范，随便交叉胡乱访问，几百个服务的话，全体乱套，这样的一套服务是没法管理的，没法治理的，可能会出现数据被别人改错，自己的库被别人写挂等情况。

如果你要操作别人的服务的库，你必须是通过**调用别的服务的接口**来实现，**绝对不允许交叉访问别人的数据库**。

![distributed-transacion-XA](./images/distributed-transaction-XA.png)

#### **潜在问题**

+ **同步阻塞，资源被锁定**
+ **单点故障**
+ **事务状态丢失（多点备份/选举），比如原来的TM发送commit消息以后挂机**
+ **脑裂**， 一个数据库挂机，不能接受消息



### 3-Pharse-Commit 改进方案

阶段一：发送canCommit消息，不会执行任何sql，只是确保资源可用

阶段二：发送preCommit消息，只是执行sql

阶段三：发送doCommit消息，执行commit

**如果一个库在preCommit执行操作，隔了一段时间没有执行doCommit消息，默认TM挂了。这时候就会自己执行commit，提交事务**。因为在第一个阶段，每个数据库都返回成功，所以TM才会发送preCommit，

+ 默认每个库都可以对canCommit都返回成功，说明资源没问题
+ 超时说明默认TM的doCommit消息，认为TM挂了，所以自己执行doCommit

#### 解决部分问题

+ 部分解决同步阻塞。如果阶段3TM挂了，那么doCommit最终还是会执行，所以阶段3不会锁定资源。阶段1和阶段2不可以。
+ 单点故障
+ 事务状态丢失（多点备份/选举），比如原来的TM发送commit消息以后挂机
+ 部分解决脑裂，在阶段3，一个数据库挂机，如果恢复最终还是可以commit



### TCC 方案
TCC 的全称是：`Try`、`Confirm`、`Cancel`。

- Try 阶段：这个阶段说的是对各个服务的资源做检测以及对资源进行**锁定或者预留**。
- Confirm 阶段：这个阶段说的是在各个服务中**执行实际的操作**。
- Cancel 阶段：如果任何一个服务的业务方法执行出错，那么这里就需要**进行补偿**，就是执行已经执行成功的业务逻辑的回滚操作。（把那些执行成功的回滚）

这种方案很少人使用，但是也有使用的场景。因为这个**事务回滚**实际上是**严重依赖于你自己写代码来回滚和补偿**了，会造成补偿代码巨大，非常之恶心。

一般来说跟**钱**相关的，跟钱打交道的，**支付**、**交易**相关的场景，我们会用 TCC，**严格保证分布式事务要么全部成功**，要么全部自动回滚，严格保证资金的正确性，保证在资金上不会出现问题。**TCC是同步方案，最好是各个业务执行的时间都比较短。**

**一般尽量别用，自己手写回滚逻辑，或者是补偿逻辑是很难维护的**。

![distributed-transacion-TCC](./images/distributed-transaction-TCC.png)



### 本地消息表
本地消息表其实是国外的 ebay 搞出来的一套思想。

+ A 系统在自己本地一个事务里操作同时，插入一条数据到消息表；
+ 接着 A 系统将这个消息发送到 MQ 中去；
+ B 系统接收到消息之后，在一个事务里，往自己本地消息表里插入一条数据，同时执行其他的业务操作，如果这个消息已经被处理过了，那么此时这个事务会回滚，这样**保证不会重复处理消息**；
+ B 系统执行成功之后，就会更新自己本地消息表的状态以及 A 系统消息表的状态；
+ 如果 B 系统处理失败了，那么就不会更新消息表状态，那么此时 A 系统会定时扫描自己的消息表，如果有未处理的消息，会再次发送到 MQ 中去，让 B 再次处理；
+ 这个方案保证了最终一致性，哪怕 B 事务失败了，但是 A 会不断重发消息，直到 B 那边成功为止。

这个方案最大的问题就在于**严重依赖于数据库的消息表来管理事务**啥的，无法应对高并发而且很难扩展

![distributed-transaction-local-message-table](./images/distributed-transaction-local-message-table.png)

### 可靠消息最终一致性方案
不要用本地的消息表了，直接基于 MQ 来实现事务。比如阿里的 RocketMQ 就支持消息事务。

+ **A 系统先发送一个 prepared 消息到 mq**，如果这个 prepared 消息发送失败那么就直接取消操作别执行了；
+ 如果这个消息发送成功过了，那么**A接着执行本地事务，如果成功就告诉 mq 发送确认confirmed消息**，如果失败就告诉 mq 回滚消息；
+ 如果发送了确认消息，那么此时 B 系统会接收到确认消息，然后执行本地的事务；
+ mq 会自动**定时轮询**所有 prepared 消息回调你的接口，**问你这个消息是不是本地事务处理失败了，所有没发送确认的消息，是继续重试还是回滚？**一般来说这里你就可以查下数据库看之前本地事务是否执行，如果回滚了，那么这里也回滚吧。这个就是避免可能本地事务执行成功了，而确认消息却发送失败了。
+ 这个方案里，要是系统 B 的事务失败了需要重试，自动不断重试直到成功，如果实在是不行，要么就是针对重要的资金类业务进行回滚，比如 B 系统本地回滚后，想办法通知系统 A 也回滚；或者是发送报警由人工来手工回滚和补偿。
+ 这个还是比较合适的，目前国内互联网公司大都是这么玩儿的，一般用 RocketMQ 支持的。

![distributed-transaction-reliable-message](./images/distributed-transaction-reliable-message.png)

### 最大努力通知方案
+ 系统 A 本地事务执行完之后，发送个消息到 MQ；
+ 会有个专门消费 MQ 的**最大努力通知服务**，这个服务会消费 MQ 然后写入数据库中记录下来，或者是放入个内存队列也可以，接着调用系统 B 的接口；
+ 要是系统 B 执行成功就 ok 了；要是系统 B 执行失败了，那么最大努力通知服务就定时尝试重新调用系统 B，反复 N 次，最后还是不行就放弃。



## 2. 技术选型

类似TCC事务的，开源框架，ByteTCC，Himly，也有一些中小型公司生产环境用了类似的分布式事务框架，知名度和普及型不高；很多公司，对类似的分布式事务，是自己写一些类似的框架

阿里开源了分布式事务框架，fescar，技术体系上有很多地方都是有自己的东西，seata，阿里开源的分布式事务框架，类似TCC事务，seata来做，这个框架是经历过阿里生产环境大量的考验的一个框架，支持dubbo、spring cloud两种服务框架，都是可以的


可靠消息最终一致性方案，ActiveMQ封装一个可靠消息服务，基于RabbitMQ封装，自己开发一个可靠消息服务，收到一个消息之后，会尝试投递到MQ上去，投递失败，重试投递。 消费成功了以后必须回调他一个接口，通知他消息处理成功，如果一段时间后发现消息还是没有处理成功，此时会再次投递消息到MQ上去，在本地数据库里存放一些消息，基于ActiveMQ / RabbitMQ来实现消息的异步投递和消费。 RocketMQ，作为MQ中间件，提供了分布式事务支持，他把可靠消息服务需要实现的功能逻辑都做好了



## 3. SEATA 实践 TCC

```bash
git clone https://github.com/seata/seata-samples.git
```

可以把seata所有的示例代码拷贝下来，里面提供的例子就是跟我们说的电商的核心例子是类似的，然后导入到IDEA中

使用脚本初始化数据库

```sql
CREATE TABLE `account_tbl` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(255) DEFAULT NULL,
  `money` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `storage_tbl` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `commodity_code` varchar(255) DEFAULT NULL,
  `count` int(11) DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `commodity_code` (`commodity_code`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `order_tbl` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(255) DEFAULT NULL,
  `commodity_code` varchar(255) DEFAULT NULL,
  `count` int(11) DEFAULT '0',
  `money` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

下载一个**seata-serve**r到本地，在这里下载：https://github.com/seata/seata/releases，然后启动起来，这是分布式事务管理中心，负责维护每一个分布式事务的状态，触发分布式事务的提交和回滚.。 

```shell
seata-server.bat -h 127.0.0.1 -p 8091 -m file
```


但是任何一个服务报错之后，seata这个分布式事务的框架会感知到，自动触发所有服务之前做的数据库操作全部进行回滚. Global Transaction 包括多个 Batched Transaction

**TC**: seater server

**TM**: 管理分布式事务状态

**RM**: 管理本地资源和分支事务， 把自己的分支事务注册到TC

![seata](../../images/seata.png)

+ TM 向 TC 请求分布式事务XID
+ TC 通知XID到 RM
+ RM 注册分支事务到 TC
+ TM 告诉 TC 触发提交请求
+ TC 告诉 RM 提交请求
  + 如果出错，TC 通知 TM
  + TM 回复 TC， TC 通知 RM 回滚

核心链路中的各个服务都需要跟TC这个角色进行频繁的网络通信，频繁的网络通信其实就会带来性能的开销，**本来一次请求不引入分布式事务只需要100ms，此时引入了分布式事务之后可能需要耗费200ms**。 网络请求可能还挺耗时的，上报一些分支事务的状态给TC，seata-server，选择基于哪种存储来放这些分布式事务日志或者状态的，file，磁盘文件，MySQL，数据库来存放对应的一些状态 


高并发场景下，**seata-server，也需要支持扩容，也需要部署多台机器**，用一个数据库来存放分布式事务的日志和状态的话，假设并发量每秒上万，分库分表，对TC背后的数据库也会有同样的压力



## 4. RocketMQ 实践 最终一致性方案

![核心交易链路](../distributed-system/images/rocketmq-transaction.png)
有些服务之间的调用是走异步的，下成功了订单之后，会通知一个wms服务去发货，这个过程可以是异步的，可以是走一个MQ的，发送一个消息到MQ里去，由wms服务去从MQ里消费消息，**RocketMQ来实现可靠消息最终一致性事务方案**

+ Producer向RocketMQ发送一个half message

+ RocketMQ返回一个half message success的响应给Producer，这个时候就形成了一个half message了，此时这个message是不能被消费的。注意，这个步骤可能会**因为网络等原因失败，可能你没收到RocketMQ返回的响应，那么就需要重试发送half message，直到一个half message成功建立为止**。

+ 接着Producer本地执行数据库操作

+ Producer根据本地数据库操作的结果发送commit/rollback给RocketMQ，如果本地数据库执行成功，那么就发送一个commit给RocketMQ，让他把消息变为可以被消费的；**如果本地数据库执行失败，那么就发送一个rollback给RocketMQ**，废弃之前的message。 注意，这个步骤可能失败，就是Producer可能因为网络原因没成功发送commit/rollback给RocketMQ
+ 如果RocketMQ自己过一段时间发现一直没收到message的commit/rollback，就回调你服务提供的一个接口。此时在这个接口里，你需要自己去检查之前执行的本地数据库操作是否成功了，然后返回commit/rollback给RocketMQ。
+ 只要message被commit了，此时下游的服务就可以消费到这个消息，此时还需要结合ack机制，下游消费必须是消费成功了返回ack给RocketMQ，才可以认为是成功了，**否则一旦失败没有ack，则必须让RocketMQ重新投递message给其他consumer**。



## 5. 其他MQ实现最终一致性

写一个可靠消息服务即可，**接收人家发送的half message，然后返回响应给人家**，如果Producer没收到响应，则重发。然后Producer执行本地事务，接着发送commit/rollback给可靠消息服务。

可靠消息服务启动一个后台线程定时**扫描本地数据库表中所有half message，超过一定时间没commit/rollback就回调Producer接口**，确认本地事务是否成功，获取commit/rollback

**如果消息被rollback就废弃掉，如果消息被commit就发送这个消息给下游服务**，或者是发送给RabbitMQ/Kafka/ActiveMQ，都可以，然后**下游服务消费了，必须回调可靠消息服务接口进行ack**

如果一段时间都没收到ack，则**重发消息给下游服务**

