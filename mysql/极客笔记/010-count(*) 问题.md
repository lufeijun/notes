# count( * ) 实现，不带任何 where 条件的
 
  myisam：把表的总行数存在磁盘上，直接返回，效率很高
  innodb：由于 mvcc 的存在，需要逐行遍历，返回统计结果，当然，如果存在非主键索引树，会优先选择，


# 解决办法

  1、使用 redis 存储数据行数
  2、使用 mysql，利用事务，保证数据准确性

# 各种 count 性能

  count( 字段 ) < count(主键) < count(1) ≈ count(*)