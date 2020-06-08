# redis 配置文件


## 说明

* ./redis-server /path/redis.conf  配置文件路径必须为第一个参数   

* 内存单位

```
1k = 1ooo bytes
1kb = 1024 bytes
1m => 1000 * 1000 bytes
1mb => 1024*1024 bytes
```

大小写不敏感



## include 

  配置文件的子文件，```include /path/local.conf``` 
  
  
## redis 模块

```loadmodule /path/xxx.so```，如果加载失败，会自动放弃。https://redis.io/modules 官方实现的模块




## NETWORK 网络部分

### bind 
只接受指定 ip 客户端的请求，为了安全  

```
bind 127.0.0.1 192.168.1.100

```

### protected-mode 保护模式


### port 指定端口


### **tcp-backlog 511**   高并发


tcp 建立连接需要经过 3 次握手，对服务器而言，会有 2 中状态，SYN_REVE 、ESTABELLISHED，所以会维护两个队列：

* 存放 SYN 的队列  半连接队列，当队列建立成功后，自动放入全连接队列，此队列数量减一
* 存放已经完成连接的队列  全连接队列，在此队列中的 tcp ，等待对应的应用程序 accept 并处理，处理一个，队列的数据减一

说明：http://iambigboss.top/post/62415_1_1.html


linux 参数：   
  somaxconn ： 全连接队列长度   
  tcp_max_syn_backlog ：  半连接长度  


实际 tcp 队列情况  

* 全连接队列长度 = min(tcp-backlog , 内核参数 net.core.somaxconn  )
* 半连接队列长度 = min(tcp-backlog , 内核参数 net.core.somaxconn, 内核参数 tcp_max_syn_backlog)



#### 参数调整
 当参数过低或者并发量过大时，会出现连接方面的错误：
 
 ```
 redis All clients disconnected... aborting
 redis Error: Connection reset by peer
 ```




### Unix socket 

 指定 socket 来监听连接请求   
 
 * unixsocket /tmp/redis.sock
 * unixsocketperm 700


### timeout time
客户端闲置 time second 之后，自动断开连接。0 表示不限制


### tcp-keepalive 300

保持 tcp 长连接，通过发送 TCP ACKs 保持客户端和服务端的链接
  






## TLS/SSL 安全


## GENERAL 常用参数



### daemonize no

是否以守护进程运行 redis

### supervised

管理 redis 守护进程的方式

* no 不监控，听天由命
* upstart 
* systemd
* auto  自动

本质上就是系统自带的进程监控程序，在 centos 系统中，设置为 systemd 时，还需要自己创建 /etc/systemd/system/redis.service 文件 

```
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/src/redis-6.0.3/src/redis-server /usr/local/src/redis-6.0.3/redis.conf
ExecStop=/usr/local/src/redis-6.0.3/src/redis-cli -a 123456  shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```

ex：如果启动失败，分别执行配置文件中的命令，很可能是命令出错导致的启动失败



### pidfile /var/run/redis_6379.pid

记录 pid 的文件


### 日志

* loglevel notice 日志级别
* logfile ”“ 日志文件路径，空表示输出到 standard output，如果为空且为守护模式，则写在 /dev/null 



### databases 16

数据库的个数， 1 ~ 15




## SNAPSHOTTING 持久化


### save 

save second changes 两者都满足时才会触发更新；  

save 900 1 900s 之后且至少有 1 个更新，从上次 save 之后
save 300 10
save 60 10000


### stop-writes-on-bgsave-error yes

当发生错误时，停止接受 write 命令


### 压缩

* rdbcompression yes
* rdbchecksum yes 文件的校验，有点消耗性能
* dbfilename dump.rdb 持久化文件的名称
* dir ./ 指定文件路径，最好直接指定绝对路径


### rdb-del-sync-files no
很少被提及


## APPEND ONLY MODE aof 持久化

### 开启 aof
```
appendonly no  该模式开启|关闭
appendfilename "appendonly.aof" 保存文件名

路径与rdb使用同一个的配置。dir=xxx
``` 

### appendfsync everysec
fsync() 函数用于告知操作系统将数据写到磁盘而不是放入缓存区，但实际情况依据操作系统而定，有的会马上将数据写到磁盘，某些则只是尽快尝试写入。

数据同步到磁盘的策略：

* no 不调用 fsync ，让操作系统决定扫描时候写到磁盘
* always 每次写入都对 append only file 进行 fsync 操作。最慢，也是最安全的选项。
* everysec：每秒进行一次 fsync 操作，这种做法

### no-appendfsync-on-rewrite no

性能调优参数。当redis在进行大量的 I/O 操作时，即使在其他线程执行 fysnc 操作，也会阻塞 redis 的同步写 write(2) 操作，所以为了缓和这种情况，可以在进行 bgsave 或者  bgrewriteaof 时禁止 fsync 操作

当 redis 的延迟过大时，可以将此参数设置为 yes，否则将其设置为 no 以保持其持久性 


### 设置 aof 文件重写的触发条件

```
auto-aof-rewrite-percentage 100  容量大小超过 100% 时才进行
auto-aof-rewrite-min-size 64mb  超过 64 mb 才进行
```

### aof-load-truncated yes
当发现 aof 文件出问题时的行为：

* yes: 尽可能的多加载数据，并且写日志记录问题
* no：redis 服务端会以错误形式终止进程。

需要使用 redis-check-aof 命令来修复 aof 文件


这个参数好像设置为 yes|no 区别不大，都不会启动 redis 服务，


### aof-use-rdb-preamble 

aof 为混合式，前半部分是 rdb 形式的文件，后边是 aof 的，这样可以加快重启进程




## REPLICATION 主从复制

连接、接受RDB、重放RDB


### replicaof <masterip> <masterport>
主机的 ip 和 端口

### masterauth <master-password>
指定密码

### masteruser <username>
针对 6.0 之后的版本，使用 ACL 权限控制

### AUTH <username> <password>.
密码


### replica-serve-stale-data yes
当从库与主库连接丢失，或者正在进行同步时  

* yes 默认值，仍然向客户端提供服务，但数据可能有问题
* no  除了部分命令，直接返回错误 SYNC with master in progress

### replica-read-only yes
是否为只读


### repl-diskless-sync no

数据同步传送方式：

* no ， Disk-backed 模式，master 子进程生成 RDB 文件，slave 接受文件
* yes   直接将 RDB 写到 socket 中，不需要在磁盘中生成文件

依据 master 的磁盘能力和网络带宽，进行不同的设置

### repl-diskless-sync-delay 5

当使用 socket 模式传送 RDB 时，延迟指定时间(s)后才开始进行传输。因为一旦开始传输，新连接过来的客户端不会受到本次更新，所有等待是为了等客户端连接



### repl-diskless-load disabled
从库重放模式

* disabled
* on-empty-db
* swapdb


### repl-ping-replica-period 10
每隔 10s ping 一下，保持活着

### repl-timeout 60
超时时间

### repl-disable-tcp-nodelay no

yes 使用少量 tcp 包，带宽也少，所以延迟会增加，默认为 no，设置的值取决于主从之间的网络情况


### repl-backlog-size 1mb
复制缓存区大小，如果在复制时，slave 端突然断开了，过了一会重新连接上之后，没必要重头开始在传递，这个值越大，表示断开的时间可以越长，这个缓存同时在 master/slave 中都存在

### repl-backlog-ttl 3600

master 端缓存失效时间，从所有 slave 都断开开始计时


### replica-priority 100

slave 优先级，值越低越好，当 master 挂掉时，redis 哨兵会选择 replica priority 值最低的作为新的 master，如果值为 0 ，则表示改 slave 不具备作为 master的条件，将永远不会被选中


### master 停止更新

* min-replicas-to-write 3
* min-replicas-max-lag 10

当某一刻时，slave 少于 3 个，如果 10 s 之后仍然少于 3 ，则 master 停止数据更新


### 附加的 slave

* replica-announce-ip 5.5.5.5
* replica-announce-port 1234






## KEYS TRACKING 键


## SECURITY 安全


## CLIENTS 客户端

### maxclients 10000
允许客户端同时连接的数量



## MEMORY MANAGEMENT 内存管理

内存管理


### maxmemory <bytes>
使用内存大小

maxmemory 1GB


### maxmemory-policy 过期策略

策略有

* volatile-lru ：从已设置过期时间的内存数据集中挑选最近最少使用的数据 淘汰
* volatile-ttl：从已经设置过期时间的内存数据中挑选即将过期的数据 淘汰
* volatile-random ：从已经设置过期时间的内存数据中任意挑选数据 淘汰
* allkeys-lru：从内存数据中挑选最近最少使用的数据 淘汰
* allkeys-random：从数据中任意挑选数据 淘汰
* no-enviction：禁止驱逐数据，

maxmemory-policy noeviction， 默认


### maxmemory-samples 5
内存超标时，选择淘汰时使用的样本数据量

* 10 算法最接近，但是毕竟耗 CPU 
* 5 比较好
* 3 算法结果不太准确


### replica-ignore-maxmemory yes

默认情况下，slave 是忽略内存限制的，做到完全由 master 触发删除 key，可以做到保持数据一致


### active-expire-effort 1

过期淘汰数据：

* 当过期的 key 被访问到时，将其删除
* 定期进行清理，避免 10% 的过期数据在内存，避免过期数据占据 25% 的内存。

这个值 1 ~ 10 ，值越大，扫描的越快，CPU 使用率上升，这是一个 CPU、内存、延迟之间的权衡，决定扫描周期





## LAZY FREEING 懒惰释放

异步删除释放 key 的内存，防止在使用 del 删除大量数据时，block redis 进程。

* 主动删除键：  
unlink 在删除集合类键时，会把真正删除的内存释放操作，给单独的 bio( background I/O ) 来操作

* 被动删除的键：  
  * lazyfree-lazy-eviction：当内存达到 maxmeory 并设置淘汰策略时，在被动淘汰时是否采用 lazy free 机制，开启后，可能导致回收内存不及时，导致 redis 内存超限制
  * lazyfree-lazy-expire：针对设置 TTL 的键，达到过期后，被 redis 清理删除时是否采用 lazy free
  * lazyfree-lazy-server-del：针对有些指令在处理已经存在的键时，会带有一个隐式的 DEL 键的操作，如 rename ，当目标键已经存在，redis 会先删除，如果这些目标键是一个 big key ，那就会引入阻塞删除的性能问题
  * slave-lazy-flush：针对 slave 进行全量数据同步，slave 在加载 master 的 RDB 文件前，会运行 flushall 来清理自己的数据场景，参数决定是否采用 lazy 机制



### lazyfree-lazy-user-del no

所有 del 是否走 unlink 命令



## THREADED I/O


## LUA SCRIPTING 脚本


## REDIS CLUSTER 集群


### cluster-enabled yes

改 redis 启动作为集群的一个节点


### cluster-config-file nodes-6379.conf

每个集群节点都需要一个集群配置文件，这个是自动产生的，但是各个节点的集群配置文件不要重名。

### cluster-node-timeout 15000
集群节点下线时长，单位时毫秒

### cluster-replica-validity-factor 10

当集群一个节点挂掉之后，挂掉节点的 slave 会进行选举，选出最新的 master 的作为集群节点。

但是在 (node-timeout * replica-validity-factor) + repl-ping-replica-period 时间之后，将不会进行选举了

这个值：  

* 设置过大，允许 slave 有很老的数据，也可以进行故障转移
* 设置过小，可能不够选举的时间

### cluster-migration-barrier 1

没太明白


### cluster-require-full-coverage yes


### cluster-replica-no-failover no


### cluster-allow-reads-when-down no






## CLUSTER DOCKER/NAT support 


## SLOW LOG 慢日志

redis 查询慢日志。redis 的慢查询只记录自己 ”被 cpu 服务的时间“，不包括排队等待、IO 等待等时间



### slowlog-log-slower-than 10000
慢查询时间，单位微妙

### slowlog-max-len 128

redis 默认保存 128 条记录，记录到慢查询日志队列中，查看

```
SLOWLOG get 2 // 获取两条慢查询记录

slowlog len // 慢查询条数

SLOWLOG RESET // 清空慢查询记录

```




## LATENCY MONITOR 监控


## EVENT NOTIFICATION 时间通知


## GOPHER SERVER


## ADVANCED CONFIG 高级配置



### hash 的编码

哈希对象的编码可以是 ziplist 或者 hashtable


hash-max-ziplist-entries 512  
hash-max-ziplist-value 64

说明：哈希对象保存的所有减至多的键和值的长度都小于 64 字节，并且保存的键值对数量少于 512 个，使用 ziplist 编码，否则使用 hashtable 编码




## ACTIVE DEFRAGMENTATION 碎片整理







# 单机 redis 配置

* 常规参数：守护模式、进程监控、pid文件、日志
* 网络部分，设置 bind 、高并发、超时问题
* 






# 配置对比项


## tcp 和 socket 

## 持久化方式





