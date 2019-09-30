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
8. 应尽量避免