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
27. 尽量使用 `>=,<=`，不要使用 `> <`。
28. 索引的使用范围：索引的创建要与应用结合考虑，建议打的OLTP 表不要超过 6 个索引；尽可能的使用索引字段作为查询条件，尤其是聚簇索引，必要时可以通过index index_name来潜在估值指定索引；避免对大表查询时进行的 table scan(表扫描) ，必要时考虑新建索引；在使用索引字段作为条件时，如果该索引是联合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则改索引将不会被使用；要注意索引的维护，周期性重建索引，重新编译存储过程。
29. 下列SQL条件中的列都建有恰当的索引，但执行速度却非常慢：

    	SELECT * FROM record WHERE substring(card_no,1,4) = '5378';(13s)
		SELECT * FROM record WHERE amount/30 <= 100 ;(11s)
		SELECT * FROM record WHERE convert(char(10),date,112) = '19991201';(10s)

分析：

WHERE 子句中对列的任何操作结果都是在SQL 运行时逐列计算得到的，因此它不得不进行表搜索，而没有使用该列上面的索引；如果这些结果在查询编译时就能得到，那么就可以被SQL优化器优化，使用索引，避免表搜索，因此将SQL重写：
		
		SELECT * FROM record WHERE card_no LIKE "5378%";(< 1s)
		SELECT * FROM record WHERE amount <= 1000*30;(< 1s)
		SELECT * FROM record WHERE date = '1999/12/01'(< 1s)

30. 当有一批处理的插入或更新时，用批量插入或批量更新，绝不会一条条记录的取更新。
31. 在所有的存储过程中，能够用SQL语句的，绝不要用循环去实现！（eg:列出上个月的每一天，挽回用connect by 去递归查询一下，绝不会去用上个月的第一天到最后一天）
32. 选择最有效率的表名顺序（只在基于规则的优化器中生效）：oracle的解析器按照从右到左的顺序处理FROM子句中的表名，FROM 子句中写在最后的表（基础表 driving table）将被最先处理，在FROM 子句中包含多个表的情况下，你必须选择记录条数最少的表作为基础表。如果有三个以上的表连接查询，那就需要选择交叉表（intersection table）作为基础表，交叉表是指那个被其他表所引用的表。
33. 提高GROUP BY 语句的效率，可以通过将不需要的记录在 GROUP BY 之前过滤掉。下面两个查询返回相同的结果，但是第二个明显快了许多。

低效：

    	SELECT  JOB,AVG(SAL) FROM EMP
		GROUP BY JOB HAVING JOB
		HAIVNG JOB = 'PRESIDENT' 
		OR JOB = "MANAGER";

高效 ： 

    	SELECT JOB,AVG(SAL) FROM EMP
		WHERE JOB = 'PRESIDENT'
		OR JOB = 'MANAGER'
		GROUP BY JOB;

34. sql 语句用大写，因为 oracle 总是先解析sql 语句，把小写的字母换成大写的再执行。
35. 别名的使用，别名是大型数据库的应用技巧，就是表名、列名在查询中以一个字母为别名，查询的速度比建链接表快1.5倍。【没太懂】
36. 避免使用临时表，除非却有需要，否则应尽量避免使用临时表，相反，可以使用变量代替；大多数时候（99%），表变量驻扎在内存中，因此速度比临时表更快，临时表驻扎在 TempDb数据库中，因此临时表上的操作需要跨数据库通信，速度自然慢。
37. 最好不要使用触发器，触发一个触发器，执行一个触发器事件本身就是一个耗费资源的过程；如果能够使用约束实现的，尽量不要使用触发器；不要为不同的触发事件（INSER,UPDATE和DELETE）使用相同的触发器；不要在触发器中使用事务型代码。
38. 避免死锁，在你的存储过程和触发器中访问同一个表时总是以相同的顺序；事务应尽可能的缩短，在一个事务中应尽量减少涉及到的数据量，永远不要在事务中等待用户输入。
39. 索引创建规则：

表主键，外检必须有索引；

数据量超过 300 的表应该有索引；

经常与其他表进行连接的表，在连接字段上应该建立索引；

经常出现在 WHERE 子句中的字段，特别是大表的字段，应该建立索引；

索引应该建在选择性高的字段上；

索引应该建在小字段上，对于大的文本字段甚至超长字段，不要建索引；

复合索引的建立需要进行仔细的分析，尽量考虑用单字段索引代替；


		