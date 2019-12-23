## MYSQL 函数 ##

    https://www.runoob.com/mysql/mysql-functions.html

### 字符串函数 ###

- ASCII(s) ： 返回字符串s的第一个字符的 ASCII 码。`SELECT ASCII(CustomerName)  AS NumCodeOfFirstChar FROM Customers;`
- CHAR_LENGTH(S) ： 返回字符串 s 的字符数。`SELECT CHAR_LENGTH("ROBY") as LengthOfString;` 
- CHARACTER_LENGTH(s) ： 返回字符串s 的字符数 `SELECT CHARACTER_LENGTH("hello") as lengthOfString;`
- CONCAT(s1,s2...sn) ： 字符串 s1,s2 等多个字符串合并为一个字符串 `SELECT CONCAT("SQL","HELLO","Google","Facebook") AS ConcatenatedString;`