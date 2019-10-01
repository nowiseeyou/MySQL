## Sql优化方案 ##

### 一、SQL语句性能优化 ###

    http://cn.voidcc.com/question/p-sgayybci-gq.html

1. 对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 以及 order by 涉及的列上建立索引。
2. 应尽量避免在 where 子句中对字段进行 null 值判断，创建表时 NULL 是默认值，但大多数使用 NOT NULL，或者使用一个特殊的值，如0，-1,作为默认值。
3. 应尽量避免在 WHERE 子句中 使用 ！= 或 <> 操作符，Mysql只有对以下操作符才使用索引 ：< ,  <= , = , > , >= , BETWEEN , IN 以及 LIKE。
4. 应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫码，可以使用 UNION 合并查询 ： `SELECT  id  FROM  table WHERE num = 10 UNION ALL select id from table WHERE num =20;`
5. in 和 not in 也要慎用，否则会导致全表扫描，对于连续的数值，能用BETWEEN 就不要用 IN 了 `SELECT id FROM t WHERE num BETWEEN 1 AND 3;` 
6. 下面的查询也将导致全表扫描： `SELECT id FROM table WHERE name LIKE '%abc%';` 或者 `SELECT id FROM table WHERE name LIKE '%abc';` 才会用到索引。
7. 如果在 where 子句中使用参数，也会导致全表扫码。
8. 应尽量避免在 where 子句中对字段进行表达式操作，应尽量避免在 where 子句中对字段进行函数操作。
9. 很多时候用 exists 代替 in 是一个好的选择 ： `SELECT num From  a WHERE num in (SELECT num FROM b);` 用下面的语句替换 ： `SELECT num FROM a WHERE exists(SELECT 1 FROM b WHERE num = a.num);`
10. 索引固然可以提高相应的 SELECT 的效率，但同时也降低了 insert 以及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建所以需要慎重考虑，视具体情况而定，一个表的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
11. 尽量避免更新 ClusteredIndex（聚合索引）数据列，因为 ClusteredIndex(聚合索引)数据列的顺序就是表记录的物理存储顺序，一单该列值改变将会导致整个表记录的顺序调整，会耗费相当大的资源。若应用系统需要频繁更新 ClusteredIndex（聚合索引）数据列，那么需要考虑是否应将该索引建为 ClusteredIndex（聚合索引）。
12. 尽量使用数字型字段，若致函数值信息的字段尽量不要设计字符型，这会降低查询和连接的性能，并会增加存储开销。
13. 尽可能的使用 varchar/nvarchat 代替 char/nchar ，因为首先边长字段存储空间小，可以节省空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
14. 最好不要使用 "" 返回所有，`SELECT FROM table`，用具体的字段列表代替  "*" ，不要返回不需要的字段。
15. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
16. 使用表的别名（alias） : 当在 SQL 语句中连接多个表时，请使用表的别名并把别名前缀于每个 Column 上，这样一来，就可以减少解析的时间并减少哪些由Column 歧义引起的语法错误。
17. 使用 "临时表" 暂存中间结果，简化SQL 语句的重要方法就是采用临时表暂存中间结果，但是临时表的好处远不止这些，将临时结果暂存在临时表，后面查询就在 tempdb 中了，这可以避免程序中多次扫描主表，也大大减少了程序执行中 "共享锁" 阻塞 "更新锁"，减少了阻塞，提高了并发性能。
18. 一些SQL 语句的应加上 nolock ，读 ， 写是会相互阻塞的，为了提高并发性能，对于一些查询，可以寄上 nolock ， 这样读的时候可以允许写，但缺点是可能督导未提交的脏数据。使用 nolock ! 3原则。查询的结果用于 "增，删，改"的不加 nolock ! 查询的表属于频繁发生页分裂的，慎用 nolock ! 使用临时表一样可以保存 "数据前影" ，起到类似 Oracle 的 undo 表空间的功能，能采用临时表提高并发性能，不要用 nolock 。
19. 常见的简化规则如下：不要超过5个以上的表连接（JOIN），考虑使用临时表或表变量存放中间结果。少用子查询，视图嵌套不要过深，一般视图嵌套不超过2个为宜。
20. 将需要查询的结果预先计算好放在表中，查询的时候再SELECT。 这在 MYSQL7.0以前是最重要的手段，例如 医院住院费计算。
21. 用 OR 的子句可以分解成多个查询，并通过 UNION 连接多个查询，他们的速度只同是否使用索引有关，如果查询需要用到联合索引，用 UNION ALL 执行的效率更高。多个OR 的子句没有用到索引，改写成 UNION 的形式再视图与索引匹配。一个关键的问题是否用到**索引**。
22. 在 IN 后面值的列表中，将出现最频繁的值放在前面，出现最少的放在最后面，减少判断的次数。
23. 尽量将数据的处理工作放在服务器上，减少网络开销，如使用存储过程。存储过程是编译好，优化过，并且被组织到一个执行规划里，且存储在数据库中的SQL语句，是控制流语言的集合，速度当然快。反复执行的动态 SQL ， 可以使用临时存储过程，该过程（临时表）被放在 tempdb中。
24. 当服务器的内存够多时，配置线程数量 = 最大连接数 +5 ，这样能发挥最大的效率，否则使用配置线程数量 < 最大连接数启用SQL SERVERD 的线程池来解决，如果还是数量 = 最大连接数 +5 ，严重的损害服务器的性能。
25. 查询的关联同写的顺序

	(A=B,B="号码")

		SELECT  a.persionMemberID,* 
		FROM chineseresume a,personmember b 
		WHERE personMemberID = b.referenceid and a.personMemberID ='JCNPRH39681';
	
	(A=B,B="号码"，A="号码")

		SELECT a.personMemberID,* FROM chineseresume a,personmember b 
		WHERE a.personMemberID = b.referenceid and a.personMemberID = "JCNPRH39681" 
		AND b,referenceid = "JCNPRH39681";

	(B = "号码" ， A = "号码")

		SELECT a.personMemberID,* FROM chinkeseresume a , personmember b
		WHERE b.referenceid = "JCNPRH39681" 
		ADN a.personMemmberID = 'JCNPRH39681';



26. 尽量使用 `exists` 代替 `SELECT COUNT(1)` 来判断是否存在记录，`count` 函数只有在统计表中所有行数时使用，而且 count(1) 比 count（*） 更有效率。


	