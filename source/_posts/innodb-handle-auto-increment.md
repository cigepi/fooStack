title: InnoDB 对 auto_increment 的处理方式
date: 2013 03 13 22:34
categories: Tech Logs
tags:
- Innodb
- MySQL
---

## 例子

```mysql
--1.首先创建一个表，包含一个auto_incremnt的id列
	CREATE TABLE case_new (
	id int(10) unsigned NOT NULL AUTO_INCREMENT,
	c_char char(10) DEFAULT NULL,
	PRIMARY KEY (id)
) ENGINE=InnoDB;
```

```mysql
--2. 使用load data语句往表中载入10行数据
mysql> LOAD DATA INFILE '/tmp/case/case_new_data.txt'
    -> INTO TABLE case_new
    -> FIELDS TERMINATED BY ','
    -> LINES TERMINATED BY '\n'
    -> (c_char);
Query OK, 10 rows affected (0.02 sec)
```

```mysql
--3. 查看当前表的auto_increment值
mysql> SELECT TABLE_NAME,AUTO_INCREMENT
    -> FROM information_schema.TABLES
    -> WHERE TABLE_NAME LIKE 'case_new';
+------------+----------------+
| TABLE_NAME | AUTO_INCREMENT |
+------------+----------------+
| case_new   |             16 |
+------------+----------------+
1 row in set (0.01 sec)
```

## innodb在内存中维护auto_increment

现在，重启MySQL服务

```console
# /etc/init.d/mysql restart
```

再次查看auto_increment值

```mysql
+------------+----------------+
| TABLE_NAME | AUTO_INCREMENT |
+------------+----------------+
| case_new   |             11 |
+------------+----------------+
```

这次对了，插入 10 条数据后的下一个`auto_increment`值应该是 11。

由此可验证：

-	**InnoDB 在内存中维护`auto_incremnet`序列，在服务重启后会依据`auto_increment`列当前最大值重新初始化。**
-	> **InnoDB** uses the in-memory auto-increment counter as long as the server runs.
	InnoDB uses the following algorithm to initialize the auto-increment counter for a table *t* that contains an **AUTO_INCREMENT** column named *ai_col*: After a server startup, for the first insert into a table *t*, **InnoDB** executes the equivalent of this statement:

	> SELECT MAX(ai_col) FROM t FOR UPDATE; 	 

	> 来自[官方文档](http://dev.mysql.com/doc/refman/5.1/en/innodb-auto-increment-handling.html)

## 不同的auto_increment处理方式

回到刚才的例子：**新表仅仅插入了 10 条数据，为何当前`auto_cremnet`值却变成了 16**

再针对该问题做一个测试，分别对新表`load`不同的数据量，观察`auto_increment`的变化规律

```text
数据量	| AUTO_INCREMENT
  1    	|       2               
  2    	|       4               
  3    	|       4               
  4    	|       8               
  5    	|       8               
  6    	|       8               
  7    	|       8               
  8    	|       16               
  15   	|       16              
  16   	|       32
```                                

可以发现，`auto_increment`的值会超出实际行数，并且取值范围为 2 的 n 次方。

**要避免这种情况，有两种方法**

1.	将参数`innodb_autoinc_lock_mode`设置为`0`（静态参数，需要重启服务，取值范围0、1、2，默认为1）。对于`innodb_autoinc_lock_mode`参数的涵义，请点击[这里](http://dev.mysql.com/doc/refman/5.1/en/innodb-auto-increment-handling.html)

2.	在编写插入语句时，让 MySQL 可以预判将要插入的总行数
	-	当使用简单`insert`语句，所有要插入内容已包含在语句内，MySQL 可以在执行前预判总行数
	-	此处使用`load data`，数据行在文件中，MySQL 无法在执行前预判总行数
	-	如果使用`insert...select`这样带子查询的语句，MySQL 无法预判总行数
	-	可以预判的，一般属于`simple inserts`语句，后面两类不能预判的一般属于`bulk inserts`

**总结起来，MySQL 对于`auto_increment`的处理，在不同`innodb_autoinc_lock_mode`时有如下不同**

-	`innodb_autoinc_lock_mode = 0`，在`simple inserts`与`bulk inserts`时，都使用表级`AUTO-INC`锁，`auto_increment`值的获取是严格串行的，可以保持最大的连续性，但性能差
-	`innodb_autoinc_lock_mode = 1`，在`simple inserts`时不使用表级`AUTO-LOCK锁`，`bulk inserts`使用表级`AUTO-LOCK`锁，连续性居中，性能居中
-	`innodb_autoinc_lock_mode = 2`，在`simple inserts`与`bulk inserts`时，都不使用表级`AUTO-INC`锁，连续性最差，性能最好
-	当`innodb_autoinc_lock_mode`为 1,2 时，对于`bulk inserts`，MySQL 可能会获得超出实际范围的`auto_increment值`
	-	> For lock modes 1 or 2, gaps may occur between successive statements because for bulk inserts the exact number of auto-increment values required by each statement may not be known and overestimation is possible.
	来自[官网文档](http://dev.mysql.com/doc/refman/5.1/en/innodb-auto-increment-handling.html)
