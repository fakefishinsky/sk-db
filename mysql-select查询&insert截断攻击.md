### 数据库字符串比较
先来看下如下两次查询：
```
mysql> select user,host from mysql.user where user = 'root';
+------+-----------+
| user | host      |
+------+-----------+
| root | localhost |
+------+-----------+
1 row in set (0.00 sec)

mysql> select user,host from mysql.user where user = 'root     '; ----后面有5个空格
+------+-----------+
| user | host      |
+------+-----------+
| root | localhost |
+------+-----------+
1 row in set (0.00 sec)

mysql> 
```
虽然2个SQL语句中使用的匹配字符串不一样，但实际查询结果相同。

这里要介绍下数据库字符串比较的原理：
在数据库对字符串进行比较时，如果两个字符串的长度不一样，则会将较短的字符串末尾填充空格，使两个字符串的长度一致，然后在进行对比。

这样就可以理解上面的2个查询结果为什么是一样的了。

### insert截断
```
mysql> create database mytest;
Query OK, 1 row affected (0.06 sec)

mysql> use mytest
Database changed
mysql>
mysql> create table user(username varchar(10), password varchar(20));
Query OK, 0 rows affected (0.21 sec)

mysql> insert into user(username, password) values('112233445566', '1234');
Query OK, 1 row affected, 1 warning (0.16 sec)

mysql> select * from user;
+------------+----------+
| username   | password |
+------------+----------+
| 1122334455 | 1234     |
+------------+----------+
1 row in set (0.00 sec)

mysql>
```
user表在创建时指定了username字段最大长度为10，而在insert插入数据时，'112233445566'长度为12，此时就会发生字符串截断，通过查询可以看到，最后的'66'没有在username中。

#### 防范措施
* 对列设置unique属性
```
mysql> alter table user add unique (username);
Query OK, 0 rows affected (0.14 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> 
mysql> desc user;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| username | varchar(10) | YES  | UNI | NULL    |       |
| password | varchar(20) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> insert into user(username,password) values('123','1234');
Query OK, 1 row affected (0.09 sec)

mysql> insert into user(username,password) values('123       abc','abcd');
ERROR 1062 (23000): Duplicate entry '123       ' for key 'username'
mysql> 
```
* sql_mode设置STRICT_TRANS_TABLES
```
mysql> set @@sql_mode=STRICT_TRANS_TABLES;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> insert into user(username,password) values('123       abc','abcd');
ERROR 1406 (22001): Data too long for column 'username' at row 1
mysql>
```
* sql_mode设置STRICT_ALL_TABLES
```
mysql> set @@sql_mode=STRICT_ALL_TABLES;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> insert into user(username,password) values('123       abc','abcd');
ERROR 1406 (22001): Data too long for column 'username' at row 1
mysql> 
```
