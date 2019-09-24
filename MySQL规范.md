## MySQL 建表规范 和 部分注意事项 ##

    https://zhuanlan.zhihu.com/p/69773290       MYSQL规范

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


### MySql避免重复插入记录方法(ignore,Replace,ON DUPLICATE KEY UPDATE) ###


**方案一：使用ignore关键字**

如果是用主键 primary或者或者唯一所有unique 区分了记录的唯一性，避免重复插入记录

    INSERT IGNORE INTO `table_name` (`email`,`phone`,`user_id`)
	VALUES ('test@qq.com','9999','999');

这样当有重复记录就会忽略，执行后返回数字0

还有个应用就是复制表，避免重复记录：

    INSERT IGNORE INTO `table_1` (`name`) SELECT `name` FROM `table_2`;

**方案二：使用Replace**

语法格式：

    REPLACE INTO `table_name` (`col_name`,...) VALUE (...);
	REPLACE INTO `table_name` (`col_name`,...) SELECT ...;
	REPLACE INTO `table_name` SET `col_name` = 'value';

算法说明：

REPLACE 的运行与INSERT 很像，但是如果旧记录与新记录有相同的值，则在新记录插入之前，旧记录被删除，即：

尝试吧新行插入到表中，当因为对于主键或唯一关键字出现重复关键字错误而造成插入失败时：

- 从表中删除含有重复关键字值的冲突行
- 再次尝试把新行插入到表中


旧记录与新记录有相同的值的判断标准就是：


- 表有一个PRIMARY KEY 或者 UNIQUE 索引，否则，使用一个 REPLACE 语句没有意义。该语句会与 INSERT 相同，因为没有索引被用于确定是否新行复制了其他行。


返回值：

REPLACE 语句会返回一个数，来指示受影响的行的数目，该数是被删除和插入的行数的和，受影响的行数可以容易的确定是否REPLACE 只添加了一行
 
