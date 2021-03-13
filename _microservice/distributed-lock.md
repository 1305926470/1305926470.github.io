---
layout: single
title:  "分布式锁"
date:   2020-12-25 11:22:26 +0800
permalink: /microservice/distributed-lock
toc: true
toc_sticky: true
---



[TOC]



## 分布式锁要求

什么时候会用到锁？并发访问。

在一个集群中，同一个方法或资源在同一时间只能被一台机器上的一个线程执行。这就是所谓的分布式互斥。大家在做某个事的时候，要去一个服务上请求一个标识。如果请求到了，就可以操作，操作完后，把这个标识还回去就把锁释放了，这样别的进程就可以请求到了。

对于分布式的锁服务，一般可以用数据库 DB、Redis 和 ZooKeeper 等实现。

无论基于那种存储设计，分布式的锁服务需要有以下几个特点。

- `排他性`，这是基本要求，在任意时刻，只有一个客户端可以获得锁。
- 唯一性，自己`加的锁，只能自己解锁`。不能被其他客户端解锁了。
- 要有`锁超时`，避免加锁节点挂了，网络故障等造成的死锁。
- 锁服务要`高可用`，只要集群大部分节点存活，Client 就可以进行加锁解锁操作。

分布式锁服务一般可以通过 Redis 和 ZooKeeper 等实现。

## 基于 redis 分布式锁

多个进程同时去设置key，存在说明已经加锁，加锁失败。如果设置key成功，则说明获得了锁。当一个客户端无法获得锁时，它应该在随机延迟后再次尝试。

对于 value 要具有随机性，最好是不会重复的，比如时间戳+客户端实例编号。解锁时还要凭此值解锁。官方推荐从 /dev/urandom 中取 20 个 byte 作为随机数。或者采用更加简单的方式，例如使用 RC4 加密算法在 /dev/urandom 中得到一个种子（Seed），然后生成一个伪随机流。

例如：

```bash
# 申请分布式锁， 值要足够随机，解锁时还要用
SET resource_name random_value NX PX 10000
```

`SET NX` 命令只会在 key 不存在的时候给 key 赋值。

`PX` 表示保存这个 key 10000ms。即`锁过期时间`



上述加锁方法的问题：

为了避免 Client 端把锁占住不释放，然后，Redis 在超时后把其释放掉。

- 如果 Client A 先取得了锁。
- 其它 客户端 ClientB 在等待 Client A 的工作完成。
- 这个时候，如果 Client A 被一个外部的阻塞调用，或是 CPU 被别的进程吃满，或是不巧碰上了 Full GC，导致 Client A 花了超过平时几倍的时间。然后，我锁服务因为怕死锁，就在一定时间超时后，把锁给释放掉了。
- 此时，Client B 获得了锁并更新了资源。
- 这个时候，Client A 服务缓过来了，然后也去更新了资源。于是乎，把 Client B 的更新给冲掉了。这就造成了数据出错。



为此我们可以引入一个版本号。并且*版本号单调递增*。

如果使用 `ZooKeeper` 做锁服务的话，那么可以使用 `zxid` 或 znode 的版本号来做这个版本号。因为 zxid 是单调自增的。

所以也有人，认为 Redis 不适合做分布式锁。有很多的边界条件。

## 版本号 乐观锁

用数据版本（Version）记录机制，即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现的。当读取数据时，将 version 字段的值一同读出，数据每更新一次，对此 version 值加一。

```sql
UPDATE table_name SET xxx = ${xxx}, version= ${version}+1 WHERE version = ${version};
```

这也是乐观锁最常用的一种实现方式。

正因为如此，`如果我们使用版本号，或是 fence token 这种方式，就不需要使用分布式锁服务了`。



**fence token**

fence token 的思路和版本号也是类似的。数据库存储 timestamp 时间截。在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则 OK，否则就是版本冲突。

```sql
SELECT stock FROM tb_product WHERE product_id= ${product_id};
UPDATE tb_product SET stock= ${new_stock} WHERE product_id= ${product_id} AND stock= ${stock};
```

先把库存数量（stock）查出来，然后在更新的时候，检查一下是否是上次读出来的库存。如果不是，说明有别人更新过了，本次 UPDATE 操作就会失败，得重新再来。



## 基于Zookeeper分布式锁

**写锁**

方案1：利用 Zookeeper 的同级节点的唯一性特性，在需要获取排他锁时，在对应的资源目录下，创建`临节点`。

最终只有一个客户端能创建成功，那么此客户端就获得了分布式锁。没有获取到锁的客户端可以在 /exclusive_lock/resource1/ 节点上注册一个子节点变更的 watcher 监听事件，以便重新争取获得锁。

```bash
[zk: localhost:2181(CONNECTED) 17] create -e /exclusive_lock/resource1/exlock 1 
Created /lock/exclusive_lock/exlock
[zk: localhost:2181(CONNECTED) 18] 
[zk: localhost:2181(CONNECTED) 18] create -e /exclusive_lock/resource1/exlock 1
Node already exists: /exclusive_lock/resource1/exlock
```



**惊群现象**

方案1实现起来简单方便，但是容易产生大名鼎鼎的*惊群现象*。

当有10个节点需要加锁时，发现锁已经被其他节持有了，如果这10个节点都监听同一个 值，当这个值发生变化时，所有监听的节点都会被唤醒。去竞争锁，但最终只会有一个节点获得锁，其他节点还得继续等待。不断重复这个换新休眠的过程。于是，我们可以使用方案2。



方式2：临有序节点特性

- 比如在 /exclusive_lock目录下，根据不同的资源设置不同的目录，/exclusive_lock/resource1，/exclusive_lock/resource2

- 当线程获取 resource1 资源的锁时，就是在 ZK /lock/resource1 目录下创建一个临时有序的节点。

- 创建节点成功后，获取 /exclusive_lock/resource1 目录下的所有临时节点，再当前创建的节点是否是序号最小的节点。

- 如果当前创建的节点是序号最小，则认为获取锁成功。

- 如果当创建的节点不是序号最小的节点，则没有获得锁，需要对节点序号的前一个`相邻节点`添加一个事件监听，当收到前面相邻节点释放锁的通知时，再尝试加锁。

```bash
[zk: localhost:2181(CONNECTED) 14] create -s -e /lock/resource1/node 1
Created /lock/resource1/node0000000003
 
[zk: localhost:2181(CONNECTED) 15] create -s -e /lock/resource1/node 1
Created /lock/resource1/node0000000004

[zk: localhost:2181(CONNECTED) 16] ls /lock/resource1
[node0000000003, node0000000004]

```



**读锁(共享锁)**

如果事务T1对数据对象O1加上了读锁，那么当前事务只能对O1进行读取操作，其他事务也只能对这个数据对象加读锁。

- 需要加锁时，在对应的资源目录下创建`临时有序节点`。

- 对于读请求，如果没有比自己序号小的子节点或比自己序号小的都是读请求，表明已经成功获取共享锁。
- 对于写请求，如果自己不是序号最小的子节点，那么就进入等待。
- 如果没有获取到共享锁，读请求向比自己序号小的最后一个写请求节点注册 watcher 监听，写请求向比自己序号小的最后一个节点注册 watcher 监听。

```
├── share_lock
│   └── resource1
│       ├── W-0000000000
│       ├── R-0000000001
│       ├── R-0000000002
│       └── W-0000000003
```



```bash
[zk: localhost:2181(CONNECTED) 23] create /share_lock/resource1 1
Created /share_lock/resource1

[zk: localhost:2181(CONNECTED) 25] create -s -e /share_lock/resource1/W- 1
Created /share_lock/resource1/W-0000000000

[zk: localhost:2181(CONNECTED) 26] create -s -e /share_lock/resource1/R- 1
Created /share_lock/resource1/R-0000000001

[zk: localhost:2181(CONNECTED) 27] create -s -e /share_lock/resource1/R- 1
Created /share_lock/resource1/R-0000000002

[zk: localhost:2181(CONNECTED) 28] create -s -e /share_lock/resource1/W- 1
Created /share_lock/resource1/W-0000000003

[zk: localhost:2181(CONNECTED) 29] ls /share_lock/resource1
[ W-0000000000, R-0000000001, R-0000000002, W-0000000003]

```



**锁超时**

设置节点是临时节点，我们的每个机器维护着一个 Zookeeper 的 Session，通过这个 Session 会话断开时，锁会被自动释放掉。

**小结**

Zookeeper 可以不需要关心锁超时时间，实现起来有现成的第三方包，如 Curator，并且支持读锁，写锁，Zookeeper 获取锁会按照加锁的顺序，所以其是公平锁。对于高可用利用 ZK 集群进行保证。同时 Zookeeper 实现了写是顺序一致性，需要完成 Leader 和 Follower 节点的数据复制才能算写入成功，性能不高，不过这也是实现高可用和一致性的代价。



## 扩展阅读

https://redis.io/topics/distlock

https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

https://zookeeper.apache.org/doc/r3.6.2/zookeeperOver.html











