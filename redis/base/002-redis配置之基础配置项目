
## daemonize no|yes
  守护进程方式运行

## pidfile
  redis 运行是，进程 id(pid) 的保存路径，默认为 /usr/local/var/run/redis.pid


## bind 127.0.0.1
  绑定主机地址

## port 6379
  端口号



## timeout 300
  当客户端闲置多长时间后自动断开，0 为关闭此功能


## loglevel notice
  记录 log 级别

## logfile
  日志文件路径

## databases 16
  数据库数量，默认为 16 ，select <dbid> , dbid 为 0 到 databases - 1



## rdbcompression yes|no
  存储数据时，数据压缩问题

## dbfilename dump.rdb
  指定本地数据库文件名

## dir 
  本地数据库文件名路径，默认 /usr/local/var/db/redis/


## requirepass ooxx
  设置 redis 链接密码


## maxclients 
  设置同一时间客户端连接上限，注释掉为不限制，

## maxmemory 
  设置最大内存限制

## include /path/to/local.conf
  把配置单独到某个独立文件





# redis 持久化问题

  持久化方式有 rdb 和 aof，两者在实际中可以组合使用


## RDB 的概念
  快照方式的进行持久化，当符合一定条件是 redis 会自动将内存中的所有数据进行快到，存到磁盘上。

  redis 会单独创建( fork ) 一个子进程来持久化，会先将数据写入临时文件，待持续化完成之后，在用这个临时文件代替上次持久化文件

  1、配置生成快照逻辑，save <seconds> <changes>
    save 900 1  # 900s 内，有一条数据改动时，就进行一次备份，save 命令可以设置多分，任何一条满足时就会触发备份
    save 300 10
    save 60 10000

  2、执行命令，手动备份，
    save：阻塞式命令，
    bgsave：后台异步执行保存快照
    lastsave：查看最后一次成功执行快照的时间

### rdb 配置的其他命令
  
  1、rdbcompression yes|no
    在进行快照时是否开启数据压缩，开启后，会额外消耗 cpu 资源，但是产生的 .rdb 文件会小

  2、dbfilename xx.rdb
    指明快照文件名
  3、dir ./
    指定快照的文件路径
  4、stop-writes-on-bgsave-error yes|no
    当写入数据到硬盘出错，是否停止保存到硬盘
  5、rdchecksum yes|no
    在存储快照之后，可以让 redis 使用 crc64 算法来进行数据校验，但是会额外消耗 10% 的性能

### rdb 优劣对比

  优： 适合大规模的数据恢复

  劣：
    1、数据完整性 和 一致性 不高
    2、在 redis 挂掉之后，可能丢失一部分数据
    3、fork 还是比较耗时的，



# aof 持久化配置

  1、开启 aof ，appendonly yes|no
  2、设置 aof 文件名称
    appendfilename “appendonly.aof” 

  3、持久化策略
    appendfsync always   同步持久化，每次有数据更新命令，就同步进行
    appendfsync everysec 异步操作，每秒记录一次
    appendfsync no 将缓存策略交给系统，

  4、aof 的重写。防止文件过大，也可以使用命令。bgrewriteaof
    触发机制：
      redis 会记录上次重写时的 aof 文件，默认配置当 aof 为上次大小的一倍，且大于 64m 时触发
      auto-aof-rewrite-percentage 10
      auto-aof-rewrite-min-size 64mb 


