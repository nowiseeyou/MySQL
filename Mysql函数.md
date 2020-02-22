## MYSQL 函数 ##

    https://www.runoob.com/mysql/mysql-functions.html

### 字符串函数 ###

- ASCII(s) ： 返回字符串s的第一个字符的 ASCII 码。`SELECT ASCII(CustomerName)  AS NumCodeOfFirstChar FROM Customers;`
- CHAR_LENGTH(S) ： 返回字符串 s 的字符数。`SELECT CHAR_LENGTH("ROBY") as LengthOfString;` 
- CHARACTER_LENGTH(s) ： 返回字符串s 的字符数 `SELECT CHARACTER_LENGTH("hello") as lengthOfString;`
- CONCAT(s1,s2...sn) ： 字符串 s1,s2 等多个字符串合并为一个字符串 `SELECT CONCAT("SQL","HELLO","Google","Facebook") AS ConcatenatedString;`
- LAST() : 函数返回指定列中最后一个记录的值 `SELECT LAST(column_name) FROM table_name;`(注释：只有 MS Access 支持 FIRST() 函数。)
- FIRST() : 函数返回指定列中第一个记录的值 `SELECT FIRST(column_name) FORM table_name;`(注释：只有 MS Access 支持 FIRST() 函数。)