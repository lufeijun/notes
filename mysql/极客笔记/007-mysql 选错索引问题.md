# 优化器逻辑
  优化器评估各个索引，找到最优的执行方案。
    判断依据有：
      1、扫描的行数、
      2、是否使用临时表、
      3、排序等各种原因综合处理
      4、其他索引树回表查询的消耗问题

  在执行之前，mysql 的 explain 会预估每条 SQL 查询扫描的行数。行数预估错误也会导致索引选择出错


### 扫描行数估计问题

  区分度：一个索引上不同的值越多，这个索引的区分度就好。一个索引上不同的值的个数，我们称之为 ”基数“(cardinality)，也就是基数越大，区分度越好

  说明：在建立索引时，索引上的基数越大，这个索引的价值就越大，查看索引的基数，show index T；





### 索引选择出错补救逻辑

  a、可能是行数估计错误导致，使用 analyze table T，重新分析表数据
  b、使用 force index( a ) 语句，强制使用某种索引
  c、修改 SQL 语句，诱导优化器选择正确的索引
  d、对索引进行审核，删除不必要的索引，或者重建更加优秀的索引











# mysql 的查询计划
  在所需要执行的 SQL 之前加 explain ，查询结果即为mysql优化器分析的结果，当遇到慢查询或者SQL性能瓶颈时，阔以使用这个进行处理

  查询结果的字段说明：

    1、id：查询序列，越大表示优先级越高
    2、select_type：查询类型
      * simple ： 简单查询
      * primary： 查询中包含任何子查询时，最外层的被标记为 primary
      * subquery：在 select 或者 where 中列表包含了自查询
      ***
    3、type：访问类型，SQL 查询优化中一个很重要的指标，
      性能有高到低
      system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > all

      * system：表中只有一条记录，很少出现
      * const：表示通过索引一次就可以找到，primary key 或者 unique 索引
      * eq_ref：唯一性索引扫描
      * ref：非唯一性索引扫描
      * fulltext：全文索引
      * ref_or_null：类似 ref，只是搜索条件包括 null 的情况，ex: where col = 2 and col is null
      * index_merge：多重范围扫描，两个表连接的每个表的连接字段上均有索引且索引有序，结果合并在一起。
      * unique_subquery：在子查询中，基于唯一索引进行扫描，类似于 eq_ref
      * index_subquery：在子查询中，基于除唯一索引之外的索引进行扫描
      * range：范围查找， in , between , > , <
      * index：只遍历索引树
      * all：遍历全表、
    4、key：实际使用的索引
    5、extra：一些额外的，但是很重要的信息
      https://www.cnblogs.com/kerrycode/p/9909093.html
    





# 问题
  1、mysql 引擎选择索引逻辑