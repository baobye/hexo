---
title: Redis安装、集群搭建
date: 2018-06-28 10:53:08
tags:
---

redis单节点的安装与启动
首先下载redis的安装包，http://download.redis.io/releases/redis-4.0.0.tar.gz

``` bash
wget http://download.redis.io/releases/redis-4.0.0.tar.gz
tar -zxvf redis-4.0.0.tar.gz 
cd redis-4.0.0
make
make install --prefix /usr/local/redis
cp redis.conf /usr/local/redis/bin
vim redis.conf 
```
<!--more-->

# 默认是no，redis server在前端进行，关闭shell之后，server进程就被终止了，修改为yes之后，成为后台守护进程

``` bash
daemonize yes
cd /usr/local/redis/bin
```

# 通过redis.conf启动redist server，不指定启动配置文件会读取默认配置

``` bash
./redis-server redis.conf
```

# 启动redis交互式命令窗口
``` bash
./redis-cli
```

# 查看redis启动状态，在交互式命令窗口下

``` bash
redis 127.0.0.1:6379> ping 
PONG
```
# 返回PONG说明redis已经安装成功。
# 其中127.0.0.1可以在redis.conf中修改bind属性，最好将其改成本机实际的IP地址，
# 例如10.4.121.202，否则在用client连接server的时候会出现问题
在/usr/local/redis/bin目录下，还有以下的命令脚本

``` bash
redis-benchamrk   # Redis性能测试工具
redis-check-aof   # AOF文件修复工具
redis-check-dump  # RDB文件检查工具
```

redis集群的部署与使用
集群的搭建依赖ruby环境，首先安装ruby的运行环境

``` bash
yum -y install ruby
yum -y install rubygems
```

# ruby和redis的接口程序
``` bash
gem install redis
```

# 如果下载不下来可以修改一下gems的源

``` bash
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
```

# 如果因为代理问题无法通过gem install的方式下载，可以直接下载.gem文件手动安装
# 下载地址 https://rubygems.org/downloads/redis-3.3.3.gem
gem install --local redis-3.3.3.gem
创建redis集群，至少需要3个主节点，若每个主节点有一个从节点，则需要6个节点

创建集群的节点，集群规划为两台物理机，6个节点：物理机10.4.121.202, 7001 7002 7003; 物理机10.4.121.203, 7001 7002 7003

``` bash
cd /usr/local
mkdir redis-cluster
cd redis-cluster
mkdir 7001 (7002 ...)

```

# 把redis安装目录bin下的文件拷贝到每个700X目录内，每台机器伪造3个节点，端口都是7001~7003

``` bash
cp /usr/local/redis/bin /usr/local/redis-cluster/7001 (7002 ...)
```

其实主要拷贝的文件如下:

``` bash
/usr/local/redis-*/src/mkreleasehdr.sh 
/usr/local/redis-*/src/redis-server 
/usr/local/redis-*/src/redis-sentinel 
/usr/local/redis-*/src/redis.conf 
```

将各个节点的redis安装完毕之后，修改每个节点的配置文件

``` bash
vim redis.conf
```

默认ip为127.0.0.1 最好改为其他节点机器可访问的ip，否则创建集群时无法访问对应的端口，无法创建集群，所有节点都在一个机器上的伪集群除外

``` bash
bind 10.4.121.202 (10.4.121.203) 
port 7001 (7002 7003) 
daemonize yes # redis后台运行
cluster-enabled true
```

其他可选的配置的属性，可不配

``` bash
pidfile  /var/run/redis_7001.pid # pidfile文件对应7001,7002,7003
cluster-config-file  nodes_7001.conf # 集群的配置  配置文件首次启动自动生成 7001,7002,7003
cluster-node-timeout  15000 # 请求超时，默认15秒
appendonly  yes # 开启aof日志，有需要就开启，每次写操作都会记录一条日志
```

分别启动每个节点的redis服务，注意，启动的时候必须使用./，像下面的命令，分两步操作。不然你会发现后台根本没有这个进程。

``` bash
cd /usr/local/redis-cluster/7001/
./redis-server redis.conf
```
检查启动情况

``` bash
ps aux | grep redis
```

启动正常，则进入redis源码目录下

``` bash
cd /usr/zhangqiang/redis-4.0.0/src
cp redis-trib.rb /usr/local/redis-cluster
cd /usr/local/redis-cluster
```

执行创建集群命令

``` bash
./redis-trib.rb create --replicas 1 10.4.121.202:7001 10.4.121.202:7002 10.4.121.202:7003 10.4.121.203:7001 10.4.121.203:7002  10.4.121.202:7003
```

如果出现下面的问题：

[ERR] Node 10.4.121.202:7001 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
将每个节点下 /usr/local/redis-cluster/7001 内的.aof、.rdb等本地备份文件删除
将新Node的集群配置文件nodes.conf删除，在/usr/local/redis-cluster/7001内。即redis.conf里面的cluster-config-file(如果提前配置过的话)
将 /usr/local/redis-cluster 内的node.conf删除(如果有的话)
10.4.121.202:7001> flushdb 清空当前数据库(可省略)
重新关闭servers，重新启动所有的节点，再次执行集群创建脚本
　　其中dump.rdb是由Redis服务器自动生成的。默认情况下，每隔一段时间redis服务器程序会自动对数据库做一次遍历，把内存快照写在一个叫dump.rdb的文件里，这个持久化机制叫做SNAPSHOT。有了SNAPSHOT后，如果服务器宕机，重新启动redis服务器程序时redis会自动加载dump.rdb，将数据库状态恢复到上一次做SNAPSHOT时的状态。

　　当出现下面的提示说明集群创建成功

>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
10.4.121.202:7001
10.4.121.203:7001
10.4.121.202:7002
Adding replica 10.4.121.203:7002 to 10.4.121.202:7001
Adding replica 10.4.121.202:7003 to 10.4.121.203:7001
Adding replica 10.4.121.202:7003 to 10.4.121.202:7002
M: 55235f44db4afc8c5d596bd09fd11a3124008af6 10.4.121.202:7001
   slots:0-5460 (5461 slots) master
M: b03994af450e48c1c1bf2a9723fcddd32a633a6e 10.4.121.202:7002
   slots:10923-16383 (5461 slots) master
S: 4ba458ca30ebf1418940433c1a2153a690840b03 10.4.121.202:7003
   replicates 8e75ff12899683e9acea855cf27f536bc02d779a
M: 8e75ff12899683e9acea855cf27f536bc02d779a 10.4.121.203:7001
   slots:5461-10922 (5462 slots) master
S: 444a7410ccff7aacc92517102891f24b50449fdf 10.4.121.203:7002
   replicates 55235f44db4afc8c5d596bd09fd11a3124008af6
S: 4ba458ca30ebf1418940433c1a2153a690840b03 10.4.121.202:7003
   replicates b03994af450e48c1c1bf2a9723fcddd32a633a6e
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join....
>>> Performing Cluster Check (using node 10.4.121.202:7001)
M: 55235f44db4afc8c5d596bd09fd11a3124008af6 10.4.121.202:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: b03994af450e48c1c1bf2a9723fcddd32a633a6e 10.4.121.202:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 4ba458ca30ebf1418940433c1a2153a690840b03 10.4.121.202:7003
   slots: (0 slots) slave
   replicates b03994af450e48c1c1bf2a9723fcddd32a633a6e
S: 444a7410ccff7aacc92517102891f24b50449fdf 10.4.121.203:7002
   slots: (0 slots) slave
   replicates 55235f44db4afc8c5d596bd09fd11a3124008af6
M: 8e75ff12899683e9acea855cf27f536bc02d779a 10.4.121.203:7001
   slots:5461-10922 (5462 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
登陆任意redis结点查询集群中的节点情况

cd /usr/local/redis-cluster/7001
./redis-cli -h 10.4.121.202 -p 7001
10.4.121.202:7001> CLUSTER INFO
# 以下是查询出的节点情况
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:5
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:1
cluster_stats_messages_ping_sent:904
cluster_stats_messages_pong_sent:991
cluster_stats_messages_sent:1895
cluster_stats_messages_ping_received:987
cluster_stats_messages_pong_received:904
cluster_stats_messages_meet_received:4
cluster_stats_messages_received:1895
Java/Scala连接redis客户端：Jedis
SBT地址：

libraryDependencies += "redis.clients" % "jedis" % "2.9.0"
连接单节点
import redis.clients.jedis.Jedis

val jedis = new Jedis("10.4.121.202", 7001)

import scala.collection.JavaConversions._

val map = new mutable.HashMap[String, String]()
map.put("name", "zhangqiang")
map.put("sex", "male")

jedis.hmset("info_zhangqiang", map)
连接池方式连接单节点
import redis.clients.jedis._

val REDIS_HOST = "10.4.121.79"
val REDIS_PORT = 6379
// redis key 过期时间，单位秒
val EXPIRE = 24 * 60 * 60
// 可用连接实例的最大数目，默认值为8；//可用连接实例的最大数目，默认值为8；
// 如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
val MAX_TOTAL = 1024
// 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8。
val MAX_IDLE = 200
// 等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException；
val MAX_WAIT = 10000
// 连接超时时间
val TIMEOUT = 30000
// 在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的；
val TEST_ON_BORROW: Boolean = true

val config = new JedisPoolConfig
config.setMaxTotal(MAX_TOTAL)
config.setMaxIdle(MAX_IDLE)
config.setMaxWaitMillis(MAX_WAIT)
config.setTestOnBorrow(TEST_ON_BORROW)

val pool = new JedisPool(config, REDIS_HOST, REDIS_PORT, TIMEOUT)

val redisClient = pool.getResource

redisClient.hset("keyName", "key", "value")
// 设置key的过期时间，单位秒
redisClient.expire("keyName", EXPIRE)
redisClient.close()

// 在程序结束时销毁连接池
lazy val hook = new Thread {
  override def run = {
    println("Execute hook thread: " + this)
    pool.destroy()
  }
}
sys.addShutdownHook(hook.run)
连接集群
import redis.clients.jedis.{HostAndPort, JedisCluster}

val jedisClusterNodes = new util.HashSet[HostAndPort]()
jedisClusterNodes.add(new HostAndPort("10.4.121.202", 7001))
jedisClusterNodes.add(new HostAndPort("10.4.121.202", 7002))
jedisClusterNodes.add(new HostAndPort("10.4.121.202", 7003))
jedisClusterNodes.add(new HostAndPort("10.4.121.203", 7001))
jedisClusterNodes.add(new HostAndPort("10.4.121.203", 7002))
jedisClusterNodes.add(new HostAndPort("10.4.121.203", 7003))
val clients = new JedisCluster(jedisClusterNodes)

import scala.collection.JavaConversions._

val map = new mutable.HashMap[String, String]()
map.put("name", "zhangqiang")
map.put("sex", "male")

clients.hmset("info_zhangqiang", map)
异常处理
异常1
redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool at
...
Caused by: java.util.NoSuchElementException: Unable to validate object at
...
原因：当设置redis的TestOnBorrow属性为true时，validate有问题就会出现此异常。异常的意思是拿不到ping成功的Redis的链接。

./redis-cli -h 10.4.121.79 -p 6379

10.4.121.79:6379> ping
(error) MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.
　　可以看到是rdb文件持久化的问题。

vim redis.conf
# 找到下面的默认配置
dbfilename dump.rdb
dir ./
　　发现存储快照的文件名是dump.rdb，执行命令的当前目录下。去到目录中发现文件都在，也没什么问题。干掉redis重启之后。再次测试。

./redis-cli -h 10.4.121.79 -p 6379

10.4.121.79:6379> ping
PONG
　　发现ping成功了。然后再重新执行程序。异常不见了~