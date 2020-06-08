# redis
  
  redis 是一个内存数据库，内存中为 key => value 格式，value 支持多种类型：字符串、列表、集合、有序集合、哈希表、流、HyperLogLog、位图
  
  
## 什么是 redis

  redis 可以理解为数据结构的服务器，客户端/服务端模式，通过命令进行交互，命令通过 TCP socket 和协议进行传送。   
  
### redis 特征：  
* 注重保存数据到硬盘中，所以 redis 快速，稳定的。 ==== 在内存保证快，保存到磁盘保证稳定
* 数据结构强调内存效率，比高级语言更省内衬
* redis 提供很多关系型数据库的特征：主从复制、持久性、集群、高可用
* 复杂版的 memcached



### redis 安装

 在源码根目录下直接 ```make``` 就行，如果需要支持 TLS ，则```make BUILD_TLS=yes```
 

<br><br>
## redis 源码的依赖包问题

  依赖包保存在 deps 目录文件中，```make``` 命令不会自动重新编译依赖库，如果依赖库有修改，需要执行 ```make distclean```
  
  
依赖库有：  

* jemalloc
* lua
* hiRedis
* linenoise

<br><br>
## 分配器 Allocator

redis 在编译的时候会指定内存分配器，可选值：libc、jemalloc、tcmalloc，Linux系统下默认为 jemalloc

* libc：标准的内存分配库 malloc 和 free
* jemalloc：Facebook 推出的
* tcmalloc：Google 推出的

redis 没有自己实现内存池，没有在标准的系统内存分配器上再加自己的东西，所以系统的内存分配器的性能及碎片会对 redis 造成性能上的影响


### jemalloc
 作为 redis 默认的内存分配器，在减少内存碎片方面做得比较好，jemalloc 在 64 位系统中，将内存划分为小、大、巨大三个范围，每个范围内又划分为了许多小的内存块单位，当 redis 存储数据时，会选择大小最合适的内存块进行存储


### MALLOC 参数
  如果需要指定内存分配器：```make MALLOC=jemalloc|libc|tcmalloc```



<br><br>
## 运行 redis
在源码根目录下 make 之后，执行    

```
./src/redis-server

./src/redis-server ./redis.conf
```


## 安装 redis
 作用是将编译后的二进制文件安装到指定目录
 
 ```
 make PREFIX=/path/directory install  # 默认为 /user/local/bin
 ```
 
 上边的命令不会配置初始化脚本和配置文件，因此有一个脚本命令可以使用
 
 ```
 ./utils/install_server.sh
 ```
 


# 文件目录介绍


* deps：除了操作系统库之外的依赖库
    * Jemalloc 内存分配器
    * hiredis：c 语言版的客户端，edis-cli, redis-benchmark and Redis Sentinel.
    * linenoise：命令行编辑库，支持自动补全，命令参数提示
    * lua：
* src：源码库，主要都是一些 C 文件源码，还有 make 之后的一些执行文件：redis-service、redis-cli、redis-sentinel、redis-benchmark
*  test：测试文件
*  utils：一些辅助的管理工具，






## 提供的命令


### 压力测试

redis-benchmark 压力测试命令，结果基本上显示每秒的处理数量，qps。
 
* -h 指定 host 127.0.0.1
* -p 指定端口 6379
* -s 服务器 socket
* -c 并发连接数 50
* -n 总的请求数 100000
* -a 密码
* -k 是否保持连接，默认会
* -r 设计 key 的随机范围，表示 key 的数量
* -P pipelining 管道功能
* -l 循环，
* -t 指定要测试的命令。-t SET,GET
* -I idle 
* -e 错误
* -q 精简输出结果
* -- csv 按照 CSV 的格式输出

```
redis-benchmar

redis-benchmark -p 6380 -q  --csv -n 1000000 -c 100 -r 1000

```






 
## 新词

* 哨兵
* 内存分配器  