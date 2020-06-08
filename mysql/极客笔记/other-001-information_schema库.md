# information_schema 库介绍

  当前数据库运行中的信息集合，许多性能分析的信息都来源此库



# 与 innodb 有关的表

  
### INNODB_TRX 当前事务运行情况表

  mysql innodb 当前正在执行的事务的信息，

  trx_id     ：  事务ID  
  trx_wight  ：  事务权重，当出现死锁现象时，权重低的事务会进行回滚操作，解决死锁问题  
  trx_state  ：  事务状态，running、lock wait 、rolling back、committing  
  trx_start  :   事务开始时间  
  <!-- 锁 -->
  trx_requested_lock_id   ： 事务正在等待的锁的id，没有时为 null；
  trx_wait_started        ： 等待锁的开始时间

  <!--  -->
  trx_mysql_thread_id     ： mysql 线程的ID ，与 show processlist 中的 id 对应
  trx_query               ： 事务正在执行的 SQL 语句  
  trx_operation_state     :  当前操作的类型，ex：fetching rows   
  trx_tables_in_use       :  当前操作使用到的表的数量
  trx_tables_locked       :  当前执行事务具有行锁的表的个数
  trx_lock_structs        ： 事务保留的锁数      /////////////////////////////////////////////////////// ????
  trx_lock_memory_bytes   :  事务的锁结构占用的内存大小  
  trx_rows_locked         ： 锁定的行数
  trx_rows_modified       ： 事务已经修改|插入的行数
  trx_concurrency_tickets :  当前事务在被换出之前可以执行多少工作  ///////////////////////////////////////// ???

  <!--  -->
  trx_isolotion_level     :  事务的隔离级别
  trx_unique_checks       :  是否为当前事务打开或关闭唯一检测     ///////////////////////////////////////// ???
  trx_foreign_key_checks  :  是否为当前事务打开或关闭外键检测     ///////////////////////////////////////// ???
  trx_last_foreign_key_error : 最后一个外键错误的详细信息         ///////////////////////////////////////// ???
  trx_adaptive_hash_latched  : 自适应哈希索引是否被锁定           ///////////////////////////////////////// ???
  TRX_ADAPTIVE_HASH_TIMEOUT  ：                               ///////////////////////////////////////// ???
  trx_is_read_only           ：值为1表示事务是只读的
  TRX_AUTOCOMMIT_NON_LOCKING ：                                ///////////////////////////////////////// ???































