---
layout: post
date:   2018-10-04 13:36:31 +0800
categories: 
---
## SQL优化
### 优化SQL的一般步骤

* 通过show status和应用特点了解各种SQL的执行频率：通过show status可以提供服务器状态信息,也可以使用mysqladmin extended-status命令获得.show status可以根据需要显示session级别的统计结果和global级别的统计结果.

* 定位执行效率较低的SQL语句：可以通过以下两种方式定位执行效率较低的SQL语句
  1.可以通过慢查询日志定位那些执行效率较低的SQL语句,用--log-slow-queries[=file_name]选项启动时,mysqld写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件.
  2.show processlist命令查看当前MySQL在进行的线程,包括线程的状态,是否锁表等等,可以实时的查看SQL执行情况,同时对一些锁表操作进行优化.

* 通过explain分析低效SQL的执行计划
* 确定问题,并采取相应的优化措施


### 常用的SQL的优化
#### 大批量插入数据:
* 对于Myisam类型的表,可以通过以下方式快速的导入大量的数据.
  ALTER TABLE tblname DISABLE KEYS;
  loading the data
  ALTER TABLE tblname ENABLE KEYS;
  这两个命令用来打开或者关闭Myisam表非唯一索引的更新.在导入大量的数据到一个非空的Myisam表时,通过设置这两个命令,可以提高导入的效率.对于导入大量数据到一个空的Myisam表,默认就是先导入数据然后才创建索引的,所以不用进行设置.
* 而对于InnoDB类型的表,这种方式并不能提高导入数据的效率.对于InnoDB类型的 表,我们有以下几种方式可以提高导入的效率;
    a.因为InnoDB类型的表是按照主键的顺序保存的,所以将导入的数据按照主键的顺序排列,可以有效的提高导入数据的效率.
    b.在导入数据前执行SET UNIQUE_CHECKS=0,关闭唯一性校验,在导入结束后执行SET UNIQUE_CHECKS=1,恢复唯一性校验,可以提高导入的效率.
    c.如果应用使用自动提交的方式,建议在导入前执行SET AUTOCOMMIT=0,关闭自动提交,导入结束后再执行SET AUTOCOMMIT=1,打开自动提交,也可以提高导入的效率.

#### 优化insert语句:
* 如果你同时从同一客户插入很多行,使用多个值表的INSERT语句.这比使用分开INSERT语句快.Insert into test values(1,2),(1,3),(1,4)...

* 如果你从不同客户插入很多行,能通过使用INSERT DELAYED语句得到更高的速度.Delayed的含义是让insert语句马上执行,其实数据都被放在内存的队列中,并没有真正写入磁盘;这比每条语句分别插入要快的多;
* 将索引文件和数据文件分在不同的磁盘上存放(利用建表中的选项);
* 如果进行批量插入,可以增加bulk_insert_buffer_size变量值的方法来提高速度,但是,这只能对Myisam表使用;
* 当一个文本文件装载一个表时,使用LOAD DATA INFILE.这通常比使用很多inset语句快20倍;
* 根据应用情况使用replace语句代替insert;
* 根据应用情况使用ignore关键字忽略重复记录.

#### 优化group by语句:
* 默认情况下,MySQL排序所有group by col1,col2,..查询的方法如同在查询中指定ORDER BY col1,col2,...
#### 优化order by语句:
* 在某些情况中,MySQL可以使用一个索引来满足order by子句,而不需要额外的排序.
#### 优化join语句:
使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成SQL操作,同时也可以避免事务或者表锁死,并且写起来也很容易.join之所以更有效率一些,是因为MySQL不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作.
#### MySQL如何优化or条件:
对于or子句,如果要利用索引,则or之间的每个条件列都必须用到索引;如果没有索引,则应该考虑增加索引.
