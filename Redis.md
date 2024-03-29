# Redis

## Redis的数据类型

Redis支持以下数据类型：

1. String（字符串）：最基本的数据类型，二进制安全，可以包含任何数据，比如jpg图片或序列化的对象，一个键最大能存储512MB。
2. Hash（哈希）：键值对集合，适合存储对象，每个 hash 可以存储 232 - 1 键值对（40多亿）。
3. List（列表）：链表结构，通过 push 和 pop 等操作从链表的两端添加、删除元素，可以通过索引下标对单个元素进行访问。
4. Set（集合）：无序集合，不允许重复元素，支持交、并、差等操作。
5. Sorted Set（有序集合）：在 Set 的基础上增加了一个排序属性，可以按照一定规则插入数据，并按照规定的规则返回数据。

每种数据类型都有相应的操作命令，可以实现各种高级应用场景，例如计数器、缓存、消息队列等。



## Redis的哨兵模式



- 💡

  Redis的哨兵模式（Sentinel）是Redis高可用性解决方案之一。哨兵可以监控Redis主节点和从节点的运行状态，当主节点出现故障时，哨兵可以自动将一个从节点升级为主节点，确保系统的可用性。

  在哨兵模式下，有若干个哨兵进程同时工作，它们会相互通信，当发现Redis主节点（也称为"master"）宕机时，会通过一定的算法选举一个新的主节点。然后，哨兵会通知客户端Redis服务已经切换到了新的主节点上，并且客户端无需做出任何更改。

  哨兵还提供了故障转移的功能。如果选举出来的新主节点也发生了故障，那么哨兵会自动执行一些列的操作，选举一个新的主节点。这种自动故障转移的功能可以大大减少手工干预，提高系统的可用性。

  在实际应用中，我们可以通过部署多个哨兵进程来提高整个Redis集群的可用性和稳定性。

  开销: 0.0152 点数

- 👤

  怎么实现的

- 💡

  Redis的哨兵模式实现主要依赖于以下两个方面：

  1. 哨兵进程：哨兵进程是一个单独的进程，它通过向Redis节点发送命令来监控节点的状态，并在节点出现故障时执行故障转移操作。每个Redis节点可以有多个哨兵进程同时监控。
  2. Sentinel集群：Sentinel集群是由多个哨兵进程组成的，它们之间通过消息通信机制来协调各自的工作。当哨兵发现主节点故障时，它会向Sentinel集群中的其他哨兵发送消息，通知它们进行选举操作。选举出新的主节点后，哨兵会向客户端发送消息，通知它们主节点已经发生变化。

  哨兵模式的具体实现流程如下：

  1. 配置Redis主从节点：首先需要启动至少2个Redis节点，其中一个为主节点，另外的都为从节点。主从节点之间通过复制机制同步数据。
  2. 部署哨兵进程：然后需要部署若干个哨兵进程，每个哨兵进程都需要配置连接到Redis集群中的主从节点。
  3. 监控节点状态：哨兵进程会周期性地向Redis节点发送PING命令，检查节点的状态。如果哨兵进程连续若干次无法收到Redis节点的响应，就会认为该节点已经失效。
  4. 发现主节点故障：当哨兵监测到主节点故障时，会向其他哨兵发送消息，进行选举操作，选举出新的主节点。
  5. 故障转移：新的主节点被选举出来后，哨兵会向客户端发送通知消息，指示客户端新的主节点地址，并进行相应的故障转移操作。

  通过以上的步骤，哨兵模式可以实现Redis高可用性的需求。

  开销: 0.02686 点数

- 👤

  有采用相似技术的框架吗

- 💡

  是的，还有一些采用类似技术的框架，比如Apache ZooKeeper和etcd。

  Apache ZooKeeper是一个开源的分布式协调服务，它提供了一个层次化的命名空间和数据存储，可以用于注册、协调和发现服务，支持分布式锁和分布式队列等功能。ZooKeeper使用Paxos算法实现无单点故障的数据复制，可以保证数据的强一致性。ZooKeeper被广泛应用于Hadoop、Kafka等分布式系统中。

  etcd是一个开源的分布式键值存储系统，由CoreOS维护，使用Raft算法实现数据的一致性，并提供API接口进行操作。etcd被广泛应用于Kubernetes等容器管理平台中，用于存储集群状态信息和配置信息。

  这些框架都是为了解决分布式系统中的一致性、可用性及容错性等问题而设计的，各自都有其适用的场景和特点。

  开销: 0.03326 点数

- 👤

  哨兵进程和Redis节点有什么区别

- 💡

  哨兵进程和Redis节点有以下区别：

  1. 功能不同：Redis节点是数据存储的主要组成部分，它实现了Redis提供的各种数据结构和命令。而哨兵进程则是负责监控Redis节点的状态，并在节点发生故障时执行故障转移操作的过程。
  2. 部署位置不同：Redis节点是实际存储数据的实体，需要部署到服务器上，而哨兵进程则可以部署在任意计算机上，通过与Redis节点通信来执行监控和故障转移操作。
  3. 工作方式不同：Redis节点在运行时接受客户端发送的请求，并返回相应结果；而哨兵进程则是定时向Redis节点发送命令来监控其状态。当节点出现故障时，哨兵进程会向其他哨兵发送消息，进行选举操作，选举出新的主节点。这样的工作方式确保了哨兵进程具有较低的性能开销和资源占用。

  总之，哨兵进程和Redis节点是两个不同的概念，哨兵进程的作用是监控和控制Redis节点的状态，确保Redis集群的高可用性和稳定性。
  
  ## 安装redis及哨兵模式部署
https://redis.io/docs/getting-started/installation/install-redis-on-linux/
1. 问题
  https://stackoverflow.com/questions/50668845/cannot-install-redis-server-on-ubuntu-18-04
```JAVA
# 修改配置文件 vi /etc/redis/redis.conf 不启用ipv6
- bind 127.0.0.1 ::1
+ bind 127.0.0.1
# 注意端口6379 没有被占用
```

2. 哨兵模式配置
https://www.cnblogs.com/kismetv/p/9609938.html
```shell
# 三个redis配置文件redis.conf

port 6380
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
-- 
port 6381
daemonize yes
logfile "6381.log"
dbfilename "dump-6381.rdb"
slaveof 127.0.0.1 6380
--
port 6382
daemonize yes
logfile "6382.log"
dbfilename "dump-6382.rdb"
slaveof 127.0.0.1 6380

# 启动三个redis
redis-server redis-6380.conf
redis-server redis-6381.conf
redis-server redis-6382.conf

# 停止redis
redis-cli -h 127.0.0.1 -p 6380 shutdown


# 连接redis
redis-cli -h 127.0.0.1 -p 6380
# 查看状态
info

####### ---------------- 3个哨兵----------------------
#sentinel-26380.conf
port 26380
daemonize yes
logfile "26380.log"
sentinel monitor mymaster 127.0.0.1 6380 2


#sentinel-26381.conf
port 26381
daemonize yes
logfile "26381.log"
sentinel monitor mymaster 127.0.0.1 6380 2


#sentinel-26382.conf
port 26382
daemonize yes
logfile "26382.log"
sentinel monitor mymaster 127.0.0.1 6380 2


启动三个sentinel 哨兵
redis-server sentinel-26380.conf --sentinel
redis-server sentinel-26381.conf --sentinel
redis-server sentinel-26382.conf --sentinel

# 连接redis 哨兵
redis-cli -h 127.0.0.1 -p 26380

# 查看状态
info sentinel
```


