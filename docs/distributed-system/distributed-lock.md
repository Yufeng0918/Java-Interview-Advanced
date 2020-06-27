# 分布式锁

一般实现分布式锁都有哪些方式？使用 redis 如何设计分布式锁？使用 zk 来设计分布式锁可以吗？这两种分布式锁的实现方式哪种效率比较高。 分布式锁在分布式系统开发中，分布式锁的使用场景还是很常见的。



## 1. Redis 分布式锁

官方叫做 `RedLock` 算法，是 redis 官方支持的分布式锁算法。

这个分布式锁有 3 个重要的考量点：

- 互斥（只能有一个客户端获取锁）
- 不能死锁
- 容错（只要大部分 redis 节点创建了这把锁就可以）

#### Redis普通的分布式锁

第一个最普通的实现方式，就是在 redis 里使用 `setnx` 命令创建一个 key，这样就算加锁。执行这个命令就 ok。


```r
SET resource_name my_random_value NX PX 30000
```

- `NX`：表示只有 `key` 不存在的时候才会设置成功。（如果此时 redis 中存在这个 key，那么设置失败，返回 `nil`）
- `PX 30000`：意思是 30s 后锁自动释放。别人创建的时候如果发现已经有了就不能加锁了。

释放锁就是删除 key ，但是一般可以用 `lua` 脚本删除，判断 value 一样才删除：

```lua
-- 删除锁的时候，找到 key 对应的 value，跟自己传过去的 value 做比较，如果是一样的才删除。
if redis.call("get",KEYS[1]) == ARGV[1]) then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

为啥要用随机值呢？因为如果某个客户端获取到了锁，但是阻塞了很长时间才执行完，**比如说超过了 30s，此时可能已经自动释放锁了，此时可能别的客户端已经获取到了这个锁，要是你这个时候直接删除 key 的话会有问题**，所以得用随机值加上面的 `lua` 脚本来释放锁。

但是这样是肯定不行的。因为如果是普通的 redis 单实例，那就是单点故障。或者是 redis 普通主从，那 redis 主从异步复制，如果主节点挂了（key 就没有了），key 还没同步到从节点，此时从节点切换为主节点，别人就可以 set key，从而拿到锁。

#### RedLock 算法
这个场景是假设有一个 redis cluster，有 5 个 redis master 实例。然后执行如下步骤获取一把锁：

+ 获取当前时间戳，单位是毫秒；
+ 跟上面类似，轮流尝试在每个 master 节点上创建锁，过期时间较短，一般就几十毫秒；
+ 尝试在**大多数节点**上建立一个锁，比如 5 个节点就要求是 3 个节点 `n / 2 + 1`；
+ 客户端计算建立好锁的时间，如果建立锁的时间小于超时时间，就算建立成功了；
+ 要是锁建立失败了，那么就依次之前建立过的锁删除；
+ 只要别人建立了一把分布式锁，你就得**不断轮询去尝试获取锁**。

![redis-redlock](../../images/redis-redlock.png)

[Redis 官方](https://redis.io/)给出了以上两种基于 Redis 实现分布式锁的方法，详细说明可以查看：https://redis.io/topics/distlock 。



#### Redisson框架加锁

Redis分布式锁，很少自己撸，Redisson框架，他基于Redis实现了一系列的开箱即用的高级功能。 比如说，苹果这个商品的id是1

```json
redisson.lock(“product_1_stock”)

product_1_stock: {
	"xxxx": 1
}
```

key的业务语义，就是针对product_id = 1的商品的库存，也就就是苹果的库存，进行加锁, 生存时间：30s

**watchdog，redisson框架后台执行一段逻辑**，**每隔10s**去检查一下这个锁是否还被当前客户端持有，如果是的话，**重新刷新一下key的生存时间为30s**

其他客户端尝试加锁，这个时候发现“product_1_stock”这个key已经存在了，里面显示被别的客户端加锁了，此时他就会陷入一个无限循环，阻塞住自己，不能干任何事情，必须在这里等待

第一个客户端加锁成功了

+ 这个客户端操作完毕之后，主动释放锁
+ 如果这个客户端宕机了，那么这个客户端的redisson框架之前启动的后台watchdog线程，就没了

此时最多30s，key-value就消失了，自动释放了宕机客户端之前持有的锁



## 2. Zookeeper 分布式锁

对某一个数据连续发出两个修改操作，两台机器同时收到了请求，但是只能一台机器先执行完另外一个机器再执行。那么此时就可以使用 zookeeper 分布式锁，一个机器接收到了请求之后先获取 zookeeper 上的一把分布式锁，就是可以去创建一个 znode，接着执行操作；然后另外一个机器也**尝试去创建**那个 znode，结果发现自己创建不了，因为被别人创建了，那只能等着，等第一个机器执行完了自己再执行。

![zookeeper-distributed-lock-demo](./images/zookeeper-distributed-lock-demo.png)

### 临时节点来获取锁

zk 分布式锁，其实可以做的比较简单，就是**某个节点尝试创建临时 znode，此时创建成功了就获取了这个锁；这个时候别的客户端来创建锁会失败，只能注册个监听器**监听这个锁。释放锁就是删除这个 znode，一旦释放掉就会通知客户端，然后有一个等待着的客户端就可以再次重新加锁。

```java
/**
 * ZooKeeperSession
 * 
 * @author bingo
 * @since 2018/11/29
 *
 */
public class ZooKeeperSession {

    private static CountDownLatch connectedSemaphore = new CountDownLatch(1);

    private ZooKeeper zookeeper;
    private CountDownLatch latch;

    public ZooKeeperSession() {
        try {
            this.zookeeper = new ZooKeeper("192.168.31.187:2181,192.168.31.19:2181,192.168.31.227:2181", 50000, new ZooKeeperWatcher());
            try {
                connectedSemaphore.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("ZooKeeper session established......");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取分布式锁
     * @param productId
     */
    public Boolean acquireDistributedLock(Long productId) {
        String path = "/product-lock-" + productId;

        try {
            zookeeper.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
            return true;
        } catch (Exception e) {
            while (true) {
                try {
                    // 相当于是给node注册一个监听器，去看看这个监听器是否存在
                    Stat stat = zk.exists(path, true);

                    if (stat != null) {
                        this.latch = new CountDownLatch(1);
                        //等待临时节点被删除
                        this.latch.await(waitTime, TimeUnit.MILLISECONDS);
                        this.latch = null;
                    }
                    zookeeper.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
                    return true;
                } catch (Exception ee) {
                    continue;
                }
            }

        }
        return true;
    }

    /**
     * 释放掉一个分布式锁
     * @param productId
     */
    public void releaseDistributedLock(Long productId) {
        String path = "/product-lock-" + productId;
        try {
            zookeeper.delete(path, -1);
            System.out.println("release the lock for product[id=" + productId + "]......");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 建立zk session的watcher
     * @author bingo
     * @since 2018/11/29
     *
     */
    private class ZooKeeperWatcher implements Watcher {

        public void process(WatchedEvent event) {
            System.out.println("Receive watched event: " + event.getState());

            if (KeeperState.SyncConnected == event.getState()) {
                connectedSemaphore.countDown();
            }

            if (this.latch != null) {
                this.latch.countDown();
            }
        }
    }

    /**
     * 封装单例的静态内部类
     * @author bingo
     * @since 2018/11/29
     *
     */
    private static class Singleton {

        private static ZooKeeperSession instance = instance = new ZooKeeperSession();

        public static ZooKeeperSession getInstance() {
            return instance;
        }
    }

    /**
     * 获取单例
     * @return
     */
    public static ZooKeeperSession getInstance() {
        return Singleton.getInstance();
    }

    /**
     * 初始化单例的便捷方法
     */
    public static void init() {
        getInstance();
    }
}
```



### 临时顺序节点获取锁

如果有一把锁，被多个人给竞争，此时多个人会排队，第一个拿到锁的人会执行，然后释放锁；后面的每个人都会去监听**排在自己前面**的那个人创建的 node 上，一旦某个人释放了锁，**排在自己后面的人就会被 zookeeper 给通知，一旦被通知了之后，就 ok 了，自己就获取到了锁，就可以执行代码了**。
```java
public class ZooKeeperDistributedLock implements Watcher {

    private ZooKeeper zk;
    private String locksRoot = "/locks";
    private String productId;
    private String waitNode;
    private String lockNode;
    private CountDownLatch latch;
    private CountDownLatch connectedLatch = new CountDownLatch(1);
    private int sessionTimeout = 30000;

    public ZooKeeperDistributedLock(String productId) {
        this.productId = productId;
        try {
            String address = "192.168.31.187:2181,192.168.31.19:2181,192.168.31.227:2181";
            zk = new ZooKeeper(address, sessionTimeout, this);
            connectedLatch.await();
        } catch (IOException e) {
            throw new LockException(e);
        } catch (KeeperException e) {
            throw new LockException(e);
        } catch (InterruptedException e) {
            throw new LockException(e);
        }
    }

    public void process(WatchedEvent event) {
        if (event.getState() == KeeperState.SyncConnected) {
            connectedLatch.countDown();
            return;
        }

        if (this.latch != null) {
            this.latch.countDown();
        }
    }

    public void acquireDistributedLock() {
        try {
            if (this.tryLock()) {
                return;
            } else {
                waitForLock(waitNode, sessionTimeout);
            }
        } catch (KeeperException e) {
            throw new LockException(e);
        } catch (InterruptedException e) {
            throw new LockException(e);
        }
    }

    public boolean tryLock() {
        try {
 		    // 传入进去的locksRoot + “/” + productId
		    // 假设productId代表了一个商品id，比如说1
		    // locksRoot = locks
		    // /locks/10000000000，/locks/10000000001，/locks/10000000002
            lockNode = zk.create(locksRoot + "/" + productId, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
   
            // 看看刚创建的节点是不是最小的节点
	 	    // locks：10000000000，10000000001，10000000002
            List<String> locks = zk.getChildren(locksRoot, false);
            Collections.sort(locks);
	
            if(lockNode.equals(locksRoot+"/"+ locks.get(0))){
                //如果是最小的节点,则表示取得锁
                return true;
            }
	
            //如果不是最小的节点，找到比自己小1的节点
	  int previousLockIndex = -1;
            for(int i = 0; i < locks.size(); i++) {
		if(lockNode.equals(locksRoot + “/” + locks.get(i))) {
	         	    previousLockIndex = i - 1;
		    break;
		}
	   }
	   
	   this.waitNode = locks.get(previousLockIndex);
        } catch (KeeperException e) {
            throw new LockException(e);
        } catch (InterruptedException e) {
            throw new LockException(e);
        }
        return false;
    }

    private boolean waitForLock(String waitNode, long waitTime) throws InterruptedException, KeeperException {
        Stat stat = zk.exists(locksRoot + "/" + waitNode, true);
        if (stat != null) {
            this.latch = new CountDownLatch(1);
            this.latch.await(waitTime, TimeUnit.MILLISECONDS);
            this.latch = null;
        }
        return true;
    }

    public void unlock() {
        try {
            // 删除/locks/10000000000节点
            // 删除/locks/10000000001节点
            System.out.println("unlock " + lockNode);
            zk.delete(lockNode, -1);
            lockNode = null;
            zk.close();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }

    public class LockException extends RuntimeException {
        private static final long serialVersionUID = 1L;

        public LockException(String e) {
            super(e);
        }

        public LockException(Exception e) {
            super(e);
        }
    }
}
```



### Curator 框架加锁

![distributed-lock](/Users/daiyu/dev/idea/architect/Java-Interview-Advanced/docs/distributed-system/images/zookeeper-distribute-lock.png)
curator基于zk实现了一整套的高级功能

```javascript
curator.lock(“product_1_stock”)
/locks/product_1_stock
```



### 羊群效应

![zookeeper-distribute-lock-optimize](/Users/daiyu/dev/idea/architect/Java-Interview-Advanced/docs/distributed-system/images/zookeeper-distribute-lock-optimize.png)
如果几十个客户端同时争抢一个锁，此时会导致任何一个客户端释放锁的时候，zk反向通知几十个客户端，几十个客户端又要发送请求到zk去尝试创建锁，所以大家会发现，几十个人要加锁，大家乱糟糟的，无序的

羊群效应, **造成很多没必要的请求和网络开销，会加重网络的负载**

### 脑裂

分布式锁脑裂，重复加锁。 分布式系统，主控节点有一个Master，此时因为网络故障，导致其他人以为这个Master不可用了，**其他节点出现了别的Master**，导致集群里有2个Master同时在运行

curator框架源码，加一些协调机制。



## 3. Redis 和 Zookeeper 分布式锁的对比

- redis 分布式锁，其实**需要自己不断去尝试获取锁**，比较消耗性能。
- zookeeper 分布式锁，**获取不到锁，注册个监听器即可，不需要不断主动尝试获取锁，性能开销较小**。不适用大集群，仅仅是分布式系统协调

如果是 redis 获取锁的那个客户端 出现 bug 挂了，那么只能等待超时时间之后才能释放锁；而 zk 的话，因为创建的是临时 znode，只要客户端挂了，znode 就没了，此时就自动释放锁。

redis 分布式锁遍历上锁，计算时间等等。**zk 的分布式锁语义清晰实现简单**。**实践中zk 的分布式锁比 redis 的分布式锁牢靠、而且模型简单易用**。



## 4. 高并发

### 分段加锁

对某个商品下单，对一个分布式锁每秒突然有上万请求过来，都要进行加锁，此时怎么办呢？ 比如你的苹果库存有10000个，

此时你在数据库中创建10个库存字段。 一个表里有**10个库存字段，stock_01，stock_02，每个库存字段里放1000个库存。** 此时这个库存的分布式锁，对应10个key，product_1_stock_01，product_1_stock_02

请求过来之后，你从10个key随机选择一个key，去加锁就可以了，每秒过来1万个请求，此时他们会对10个库存分段key加锁，每个key就1000个请求，每台服务器就1000个请求而已。


万一说某个库存分段仅仅剩余10个库存了，此时我下订单要买20个苹果，合并扣减库存，你对product_1_stock_5，加锁了，此时查询对应的数据库中的库存，此时库存是10个，不够买20个苹果。 你可以尝试去锁product_1_stock_1，再查询他的库存可能有30个

此时你就可以下订单，锁定库存的时候，就对product_1_stock_5锁定10个库存，对product_1_stock1锁定10个库存，锁定了20个库存


分段加锁 + 合并扣减



### 分布式KV存储

大公司一般有**分布式kv存储**，tair，redis，mongodb，高并发，每秒几万几十万都没问题，甚至每秒百万

实时库存数据放kv存储里去，先查库存再扣减库存，你在操作库存的时候，直接扣减，如果你发现扣减之后是负数的话，此时就认为库存超卖了，回滚刚才的扣减，返回提示给用户。对kv做的库存修改写MQ，异步同步落数据库，相当于异步双写，用分布式kv抗高并发，做好一致性方案