## MySQL ##

- 常用字段放在表的前几列，不用的字段放在表的后面几列。
- 查询时 每个字段加上表名，单表也需要加的。因为MySQL优化器，在执行SQL的时候它会把表名自动加上的。如果我们做了这一步，优化器就不会做这一步了。
- 使用locate函数或者position函数代替like查询：
如table.field like '%AAA%'可以改为locate('AAA', table.field) > 0或POSITION('AAA' IN table.field)>0