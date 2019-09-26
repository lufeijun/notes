# mysql 的 for update



## 使用场景
  在高并发场景下，对数据准确有严格的要求，就需要加锁，防止其他线程进行更新。
  ex: 库存秒杀活动，库存为 1 后，同时有俩线程进来，查询值为1，如果都进行减一的话，这样会导致数据出错。

## 语句
  select * from table where .... for update。

## 解释说明
  在 innodb 中，默认的锁是行级锁，当有明确的主键时候，是行级锁，否则，会升级为表锁

  各种情况：
    1、明确指定主键|索引，并且有记录，行级锁
      select * from table where id = XXX for update；

    2、明确指定主键|索引，但是没有记录，会加一个间隙锁
      select * from table where id = 0 for update；

    3、无主键|索引，表级锁
      select * from table where name = "ooxx" for update；      

    4、主键|索引不明确，表级锁
      select * from table where id > XXX for update；      

    5、for update 仅适用于 innodb ，并且需要开启事务，


# 数据幻读问题

  四种隔离级别
    a、读未提交 read-uncommitted|0 ：存在读脏，不可重复读、幻读问题
    b、读提交   read-committed|1   ： 解决了读脏问题，但存在不可重复读、幻读问题
    c、可重复读 repeatable-read|2  ：解决读脏、不可重复读问题，存在幻读问题，默认隔离级别，使用 mmvc 机制实现可重复读；
    d、序列化读 serializable|3 : 解决读脏、不可重复读、幻读问题，但执行为串行化，性能最低。

  一般数据库都设置为 可重复读 隔离级别；

## 概念
  某一次的 select 操作得到的结果无法支持后续业务操作，ex：select 某记录是否存在，不存在，准备执行 insert 操作时，发现另一个线程执行了 insert 操作，导致问题发生。

## 解决
  在 可重复读 隔离级别下，解决幻读的情况就是加锁( for update )，
  InnoDB 的行锁锁定的是索引，而不是记录本身，这一点也需要有清晰的认识，故某索引相同的记录都会被加锁，会造成索引竞争，这就需要我们严格设计业务sql，尽可能的使用主键或唯一索引对记录加锁。
  索引映射的记录如果存在，加行锁，如果不存在，则会加 next-key lock / gap 锁|间隙锁，故 InnoDB 可以实现事务对某记录的预先占用，如果记录存在，它就是本事务的，如果记录不存在，那它也将是本是无的，
  只要本是无还在，其他事务就别想占有它。