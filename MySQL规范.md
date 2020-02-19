## MySQL 建表规范 和 部分注意事项 ##
	//大数据下的 limit 性能消耗过大
	https://www.eversql.com/faster-pagination-in-mysql-why-order-by-with-limit-and-offset-is-slow/
 
	//MYSQL规范
    https://zhuanlan.zhihu.com/p/69773290 

	//Mysql 总结
	https://www.kancloud.cn/digest/mysqlsummary/132847      

- 常用字段放在表的前几列，不用的字段放在表的后面几列。
- 查询时 每个字段加上表名，单表也需要加的。因为MySQL优化器，在执行SQL的时候它会把表名自动加上的。如果我们做了这一步，优化器就不会做这一步了。
- 使用locate函数或者position函数代替like查询：
如table.field like '%AAA%'可以改为locate('AAA', table.field) > 0或POSITION('AAA' IN table.field)>0


    	SELECT count(CASE WHEN parentId=932 THEN 1 END) 
		AS count1,count(CASE WHEN zparentId=932 THEN 1 END) 
		AS count2,count(CASE WHEN gudongId=932 THEN 1 END)
		AS count3 FROM ssc_members 
		WHERE regTime>=1561910400



**注意：**
避免数据类型的隐式转换
隐式转换会导致索引失效。如：

	select name,phone from customer where id = '111';

select 查询 时 可以为字段指定强制索引

    SELECT * FROM table force index(filed) where id > 1;
 
### 数据库命令规范 ###

- 所有的数据库对象名称必须使用小写字母并用下划线分隔
- 所有的数据库对象明细禁止使用MySQL保留关键字（如果表名中包含关键字查询时，需要将其用哪个单引号括起来）
- 数据库对象的命名要能做到见名识意，并且最后不要超过12个字符
- 临时库表必须以tmp_为前缀并以日期为后缀，备份表必须以bak_为前缀并以日期（时间戳）为后缀
- 所有存储相同数据的列名和列类型必须一致（一般作为关联列，如果查询时关联列类型不一致会自动进行数据类型的隐式转换，会造成列上的索引失效，导致查询效率降低）

### Explain ###

MySQL 提供了一个 EXPLAIN 命令, 它可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.

### 笛卡尔积 ###

现在，我们有两个集合A和B。

A = {0,1}     B = {2,3,4}

集合 A×B 和 B×A的结果集就可以分别表示为以下这种形式：

A×B = {（0，2），（1，2），（0，3），（1，3），（0，4），（1，4）}；

B×A = {（2，0），（2，1），（3，0），（3，1），（4，0），（4，1）}；

以上A×B和B×A的结果就可以叫做两个集合相乘的‘笛卡尔积’。

从以上的数据分析我们可以得出以下两点结论：

1. 两个集合相乘，不满足交换率，既 A×B ≠ B×A;

2. A集合和B集合相乘，包含了集合A中元素和集合B中元素相结合的所有的可能性。既两个集合相乘得到的新集合的元素个数是 A集合的元素个数 × B集合的元素个数;

## 分表 ##
	$month = date('Ym');
	// table_name 基础表 
    CREATE TABLE IF NOT EXISTS ".$cur_table.'_'.$month." LIKE table_name;


## Mysql事务和锁 SELECT FOR UPDATE ##

    https://zhuanlan.zhihu.com/p/53974502

### 事务 ###

当然有的人用 begin/begin work .推荐使用 START  TRANSACTION 是 SQL -99 标准启动一个事务。

    start transaction 	# 开始一个事务
	操作	
	savepoint sp1 		# 保存点名称
	操作
	ROLLBACK
	ROLLBACK TO sp1		# 回退到 sp1点
	
	commit				# 提交


当用 set autocommit = 0 的时候，你以后的所有的sql都将作为事务处理，直到你用`commit`确认或`rollback`结束，注意当你结束这个事务的同时也开启了新的事务！MySQL默认 autocommit = 1,是自动提交的。

### 隔离级别 ###

SQL 标准定义的四个隔离级别为：

1. 读未提交（Read Uncommitted） : 在READ COMMMITED 的事务隔离级别下，除了唯一性的约束检查以及外键约束的检查需要 Gap Lock, InnoDB 存储引擎不会使用Gap Lock 的锁算法。这种隔离级别可以让当前事务读取到其他事务还没有提交的数据。这种读取应该是在回滚段中完成的。通过上面的分析，这种隔离级别是最低的，会导致引发脏读，不可重复读，和幻读。
2. 读已提交（Read Committed）: 这种隔离级别可以让当前事务读取到其他事务已经提交的数据。通过上面的分析，这种隔离级别会导致引发不可重复读，幻读。
3. 可重复读取（Repeatable Read）: 这种隔离级别可以保证在一个事务中多次读取特定记录的时候 都是一样的。通过上面的分析，这种隔离级别会导致引发幻读。
4. 串行（Serializable）：在SERIALIZBLE的事务隔离级别，InnoDB存储引擎会对每个SELECT语句后自动加上 LOCK IN SHARE MODE ,即给每个读取操作加一个共享锁，因此在这个事务隔离级别下，读占用锁了，一致性的非锁定读不在予以支持，一般不在本地事务中使用 SERIALIZBLE 的隔离级别，SERIALIZABLE 的事务给力级别主要用于InnoDB存储引擎的分布式事务。



		// 查看当前会话的事务隔离级别
    	mysql> select @@tx_isolation;

		// 查看全局事务隔离级别
		mysql> select @@global.tx_isolation;

### 分布式事务 ###

通过 XA 事务可以来支持分布式事务的实现，在使用分布式事务时，InnoDB存储引擎必须使用 SERIALIZABLE 的隔离级别，查看是否启用了 XA 事务支持（默认开启）。

    mysql> show variables like 'innodb_support_xa';

### mysql 悲观锁 ###

	




	
