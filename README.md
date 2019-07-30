# Apache Curator

## What's is Apache Curator?

Apache Curator is a Java/JVM client library for Apache ZooKeeper[1], a distributed coordination service.

Apache Curator includes a high-level API framework and utilities to make using Apache ZooKeeper much easier and more reliable. It also includes recipes for common use cases and extensions such as service discovery and a Java 8 asynchronous DSL.
For more details, please visit the project website: http://curator.apache.org/

[1] Apache ZooKeeper https://zookeeper.apache.org/

## Curator
+ 当前版本：4.0.2

## 节点概念

    PERSISTENT：持久化的节点。一旦创建后，即使客户端与zk断开了连接，该节点依然存在。
    PERSISTENT_SEQUENTIAL：持久化顺序编号节点。比PERSISTENT节点多了节点自动按照顺序编号。
    EPHEMERAL：临时节点。当客户端与zk断开连接之后，该节点就被删除。
    EPHEMERAL_SEQUENTIAL：临时顺序编号节点。比EPHEMERAL节点多了节点自动按照顺序编号。（分布式锁实现使用该节点类型）



## 分布式锁：
+ 实现逻辑：
```$xslt
1、A、B两个线程同时加锁，A先获得锁，在zk上创建一个临时顺序节点/curator_lock/锁标识-0000000000，同时会获取/curator_lock下的所有节点，
因为节点是有序的，如果A创建的节点排在第一个，那么A就获得了锁。
2、B也按照同样的方式创建一个临时顺序节点/curator_lock/锁标识-0000000001，接着也获取所有节点，但是B的节点不是最小的。所以B在前一个节点上加上一个监听器。
只要A锁释放了，B就获取到锁了
```

+ 代码：
```$xslt
CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2182",
                5000,10000,
                new ExponentialBackoffRetry(1000, 3));
        client.start();
        InterProcessMutex interProcessMutex = new InterProcessMutex(client, "/curator_lock");
        //加锁
        interProcessMutex.acquire();
        
        //业务逻辑
        
        //释放锁
        interProcessMutex.release();
        client.close();
        
```
+ curator实现代码：
```$xslt
String lockPath = internals.attemptLock(time, unit, getLockNodeBytes());
if ( lockPath != null )
{
    LockData newLockData = new LockData(currentThread, lockPath);
    threadData.put(currentThread, newLockData);
    return true;
}

锁的所有逻辑都是在attempLock里面，如果加锁成功，会返回lockPath，否则返回的是null
```