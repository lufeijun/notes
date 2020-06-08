# mysql 数据 crash 能力

  只要 redo log 和 binlog 保证持久化到磁盘，就能保证 mysql 异常重启后，数据仍然可以恢复



# binlog 写入机制

  事务提交过程中，先把日志写到 binlog cache ，事务提交的时候，再把 binlog cache 中的内容写到 binlog 文件中，

  
  1、mysql 执行事务时，binglog 日志先写到 binlog cache 中；
  2、在 commit 后，执行器把 binlog cache 中的完整事务写到 binlog 中，并清空 binlog cache，这一步本质是调用系统 write 函数，把日志写到
     文件系统的 page cache 中，实际上并没有把日志持久化到磁盘
  3、系统 fsync ，才是真正的，把数据持久化到磁盘的操作。所以，一般情况下，fysnc 才占用磁盘的 iops。







# redo log 写入机制

  事务执行过程中，生成的 redo log 先写到 redo log buffer中，在 commit 后，统一写到 redo log 文件

  redo log buffer 是所有线程公用的，所以，事务 A 的redo log 可能被 事务B 提交 而写入 redo log 文件。、、、、完全靠状态去判断么？？？？？


  redo log 三种状态：
    1、redo log buffer 中，物理上是mysql 的进程内存  
    2、调用 write，写到 page cache 中  
    3、持久化到磁盘，hard disk；  




# 双 1 配置
  sync_binlog、innodb_flush_log_at_trx_commit 都设置为 1 ，所以每次事务提交，需要等待两次磁盘刷盘，一次 redo log，一次 binlog


## 组提交机制
  组团提交，减少磁盘写次数
  












# 参数

  1、binlog_cache_size ： 
    mysql 给每个线程分派了一片内存，由此参数指定内存大小，如果超过了这个值，就要暂存到磁盘

  2、sync_binlog ： 控制 write 和 fsync 的时间
     值为 0 时：每次提交事务，只 write ，不 fsync 
     值为 1 时，每次提交事务都会执行 fsync
     值为 N 时，表示每次提交事务都会 write ，但是累计 N 个事务后才会 fsync ， N > 1

    因此，在出现 IO 性能问题时，可以将 fysnc_binlog 设置为一个较大的值。在实际业务中，考虑到数据丢失，一般设置为 100 ~ 1000 中的某个数值

    对于值为 N ，mysql 异常重启没什么影响，但是系统主机如果发生异常重启，则会导致 binlog 日志丢失，fsync 还没来得及真正的持久化，对数据恢复和主从有影响


  3、innodb_flush_log_at_trx_commit
    值为 0 ：每次事务提交只是把 redo log 写到 redo log buffer 中
    值为 1 ：每次提交事务，都将 redo log 持久化到磁盘
    值为 2 ：每次事务提交时，只把redo log 写到 page cache 中
    
    innodb 有个后台线程，每隔 1 秒 会自动把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache ，然后调用 fsync ，持久化到磁盘
  

  4、innodb_log_buffer_size : redo log buffer 的内存大小
    
     当 redo log buffer 占用的空间达到 innodb_log_buffer_size 一半的时候，后台线程会主动写盘















