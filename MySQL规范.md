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