---
layout: single
title:  "Zookeeper"
date:   2020-12-25 11:22:26 +0800
permalink: /microservice/zookeeper
toc: true
toc_sticky: true
---



[TOC]



## 安装

需先安装Java，配置环境变量，例如

```
export JAVA_HOME=/opt/jdk1.8.0_181
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=./://$JAVA_HOME/lib:$JRE_HOME/lib
```

下载对应的zookeeper二进制文件，
`wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz`

解压到/usr/local/zookeeper ， 添加环境变量：

```
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

修改 ZooKeeper 的配置文件

```
cd $ZOOKEEPER_HOME/conf
cp zoo_sample.cfg zoo.cfg
# 修改 zoo.cfg 配置文件
# ZooKeeper服务器心跳时间，单位为ms
tickTime=2000
# 投票选举新leader的初始化时间
initLimit=10
# leader与follower心跳检测最大容忍时间，响应超过syncLimit*tickTime，leader认为
# follower“死掉”，从服务器列表中删除follower
syncLimit=5
# 数据目录
dataDir=/tmp/zookeeper/data
# 日志目录
dataLogDir=/tmp/zookeeper/log
# ZooKeeper对外服务端口
clientPort=2181
```

创建这两个目录

```
 mkdir -p /tmp/zookeeper/data
```

在${dataDir}目录（也就是/tmp/zookeeper/data）下创建一个 myid 文件,并写入一个数值, 假如集群模式有3个节点，则在myid中写入1, 2, 3数字代表服务器的编号。需要唯一。

添加集群配置到  zoo.cfg文件；格式为 server.A=B:C:D。其中A是一个数字，代表服务器的编号，就是前面所说的 myid 文件里面的值。B代表服务器的IP地址。C表示服务器与集群中的 leader 服务器交换信息的端口。D表示选举时服务器相互通信的端口。

```
server.1=192.168.33.11:2888:3888
server.2=192.168.33.12:2888:3888
server.3=192.168.33.13:2888:3888
```

配置完成。常用命令：

1. 启动ZK服务:       zkServer.sh start
2. 查看ZK服务状态:   zkServer.sh status
3. 停止ZK服务:       zkServer.sh stop
4. 重启ZK服务:       zkServer.sh restart

注：注意防火墙开放端口.



## ZK

ZooKeeper 是一个集中的服务，主要用于分布式配置信息的维护、命名注册、分布式同步和中心化服务。

ZooKeeper 包含一系列的节点，叫做 Znode，就像文件系统一样，每个 Znode 表示一个目录。

ZooKeeper 节点将它们的数据存储于一个分层的命名空间，非常类似于一个文件系统或一个前缀树结构。客户端可以在节点读写，从而以这种方式拥有一个共享的配置服务。

在一致性要求上， Zookeeper 通过 ZAB 协议实现了写操作的顺序一致性。

- 所有写操作都是在主节点上进行，通过原子广播将数据同步到跟随节点上。
- 读操作可以再任意节点上进行。

当前 ZooKeeper  有如下四种事件：

- 节点创建
- 节点删除
- 节点数据修改
- 子节点变更

ZooKeeper 可以创建4种类型的节点（通过 -s 和 -e 组合控制）：

- 持久性节点
- 持久性顺序节点
- 临时性节点
- 临时性顺序节点

## 操作

主要的操作有：

```
	stat path [watch]
	set path data [version]
	ls path [watch]
	delquota [-n|-b] path
	ls2 path [watch]
	setAcl path acl
	setquota -n|-b val path
	history 
	redo cmdno
	printwatches on|off
	delete path [version]
	sync path
	listquota path
	rmr path
	get path [watch]
	create [-s] [-e] path data acl
	addauth scheme auth
	quit 
	getAcl path
	close 
	connect host:port
```



### create 命令

创建节点并赋值。

```
create [-s] [-e] path data acl
```

- **[-s] [-e]**：-s 和 -e 都是可选的，-s 代表顺序节点， -e 代表临时节点， -s 和 -e 可以同时使用的，并且临时节点不能再创建子节点。
- **path**：指定要创建节点的路径。
- **data**：要在此节点存储的数据。
- **path**：访问权限相关，默认是 world，相当于全世界都能访问。

```bash
[zk: localhost:2181(CONNECTED) 27] create /server 1   
Created /server

[zk: localhost:2181(CONNECTED) 4] create -s /lock 1
Created /lock0000000002

[zk: localhost:2181(CONNECTED) 17] create /lock0000000002/node1 1
Created /lock0000000002/node1
```

**节点有序**

当只用 -s 选项时，表示节点有序，ZK 在生成子节点时会根据当前的子节点数量自动添加整数序号。

```bash
zk: localhost:2181(CONNECTED) 32] ls /
[server, zookeeper, lock0000000003, lock0000000002] 

[zk: localhost:2181(CONNECTED) 33] create -s /node- 1 
Created /node-0000000006

[zk: localhost:2181(CONNECTED) 34] ls /
[server, node-0000000006, zookeeper, lock0000000003, lock0000000002]

[zk: localhost:2181(CONNECTED) 35] create -s /node 12138
Created /node0000000007

[zk: localhost:2181(CONNECTED) 36] ls /                 
[server, node-0000000006, zookeeper, lock0000000003, node0000000007, lock0000000002]
```



**临时节点**：通过 `-e` 建立一个临时节点，在会话结束或者会话超时后，ZK 会自动删除该节点。临时节点不能创建子目录。

```bash
[zk: localhost:2181(CONNECTED) 37] create -e /tmp 1
Created /tmp

[zk: localhost:2181(CONNECTED) 39] create /tmp/node 11
Ephemerals cannot have children: /tmp/node

```



### stat 命令

查看节点状态信息。

```
stat path [watch]
```

- **path**：代表路径。
- **[watch]**：对节点进行事件监听。

```bash
[zk: localhost:2181(CONNECTED) 9] create -s /lock 2     
Created /lock0000000003
[zk: localhost:2181(CONNECTED) 10] create -s /lock 4
Created /lock0000000004
[zk: localhost:2181(CONNECTED) 11] ls /
[zookeeper, lock0000000004, lock0000000003, lock0000000002]
[zk: localhost:2181(CONNECTED) 12] stat /lock000000000

lock0000000004   lock0000000003   lock0000000002
[zk: localhost:2181(CONNECTED) 12] stat /lock0000000004
cZxid = 0x10000000a
ctime = Sun Feb 07 10:15:48 CET 2021
mZxid = 0x10000000a
mtime = Sun Feb 07 10:15:48 CET 2021
pZxid = 0x10000000a
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
```

### get 命令

get 命令用于获取节点数据和状态信息。

格式：

```
get path [watch]
```

- **path**：代表路径。
- **[watch]**：对节点进行事件监听。

```bash
[zk: localhost:2181(CONNECTED) 4] get /Singapore
12138
cZxid = 0x100000014
ctime = Sun Feb 07 10:54:21 CET 2021
mZxid = 0x100000014
mtime = Sun Feb 07 10:54:21 CET 2021
pZxid = 0x100000014
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

事件监听：在读取数据时，可以同时对节点设置事件监听，当节点数据或结构变化时，ZK 会通知客户端。



```bash
# 客户端1开启监听
[zk: localhost:2181(CONNECTED) 5] get /Singapore watch
...
# 另一个客户端修改，
zk: localhost:2181(CONNECTED) 41] set /Singapore 2288

# 客户端1收到通知
[zk: localhost:2181(CONNECTED) 6] 
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/Singapore

# 值已修改
[zk: localhost:2181(CONNECTED) 6] get /Singapore      
2288
cZxid = 0x100000014
ctime = Sun Feb 07 10:54:21 CET 2021
mZxid = 0x100000015
mtime = Sun Feb 07 10:57:42 CET 2021
pZxid = 0x100000014
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
```



### set 命令

set 命令用于修改节点存储的数据。

格式：

```
set path data [version]
```

- **path**：节点路径。
- **data**：需要存储的数据。
- **[version]**：可选项，版本号(可用作乐观锁)。



### delete 命令

于删除某节点。

```
delete path [version]
```

- **path**：节点路径。
- **[version]**：可选项，版本号（同 set 命令）。

```bash
delete /lock0000000004
```



### ls ls2 命令

1、查看某个路径下目录列表。

```
ls /
```

ls2 命令用于查看某个路径下目录列表，它比 ls 命令列出更多的详细信息。

```bash
[zk: localhost:2181(CONNECTED) 3] ls2 /zookeeper
[quota]
cZxid = 0x0
ctime = Thu Jan 01 01:00:00 CET 1970
mZxid = 0x0
mtime = Thu Jan 01 01:00:00 CET 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```



## 基于Zookeeper分布式锁

**写锁**

方案1：利用 Zookeeper的同级节点的唯一性特性，在需要获取排他锁时，在对应的资源目录下，创建`临节点`。

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

设置节点是临时节点，我们的每个机器维护着一个 ZK 的 Session，通过这个 Session，ZK 可以判断机器是否宕机。



优点：ZK 可以不需要关心锁超时时间，实现起来有现成的第三方包，如 Curator，并且支持读写锁，ZK 获取锁会按照加锁的顺序，所以其是公平锁。对于高可用利用 ZK 集群进行保证。

缺点：ZK 需要额外维护，增加维护成本，性能和 MySQL 相差不大，依然比较差。



