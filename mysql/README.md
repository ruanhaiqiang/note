# MYSQL 学习笔记
2014-01-16 20:27

## mysql 帮助信息查看
```sql

mysql> help contents
-- 或
mysql> ? contents

You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility
```


* SQL语句主要分为3类

    * DDL (Data Definition Languages) 数据定义语句, 常用语句关键字如: create, drop, alert.
    * DML (Data Manipulation Languages) 数据操纵语句，常用语句关键字如: insert, delete, update, select.
    * DCL (Data Control Languages)  数据控制语句， 控制不同数据的访问级别。常用语句关键字如: grant, revoke.

* 常用的DDL语句

    * 创建表

    <pre>

    CREATE TABLE tablename (
            column_name_1 column_type1 constraints,
            column_name_2 column_type2 constraints,
            ...
            )

    </pre>

    * 删除表

    <pre>
    DROP TABLE tablename
    </pre>

    * 修改表

        1. 修改表类型

        ```sql
        -- 语法
        ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]

        -- 将emp表ename列类型修改为varchar(20)
        ALTER TABLE emp modify ename varchar(20);
        ```

        2. 更多操作

        ```sql
        mysql> help  alter table
        ```

* 常用的DDL语句

```sql

-- 赋予权限
mysql> grant select, insert on dabasename.* to 'user1'@'localhost' indentified by 'password';

-- 移除权限
mysql> revoke select, insert on dabasename.* to 'user1'@'localhost' indentified by 'password';
```

## mysql信息查看

* 显示创建表的SQL语句

```sql
mysql> show create table table_name
```

* 显示表列

```sql
mysql> show columns from table_name

-- 等价于

mysql> desc table_name
```

* 显示服务器的状态信息

```sql
mysql> show status
```

* 显示创建特定数据库或表的语句


```sql
mysql> show create database database_name;
mysql> show create table table_name;
```

* 显示服务器的错误和警告信息

```sql
mysql> show errors
mysql> show warnings
```

* 更多关于show命令的

```sql
mysql> help show
```


## 查询数据

* DISTINCT，返回不同的值。

注意： 不能部分使用DISTINCT， DISTINCT关键字作用于所有列而不仅是它的前置列，如果给出
SELECT DISTINCT vend_id, prod_price，除非制定的两个列都不同，否则所有的行都将被查询出来

```sql
mysql> select distinct vend_id, prod_price from products;
+---------+------------+
| vend_id | prod_price |
+---------+------------+
|    1001 |       5.99 |
|    1001 |       9.99 |
|    1001 |      14.99 |
|    1003 |      13.00 |
|    1003 |      10.00 |
|    1003 |       2.50 |
|    1002 |       3.42 |
|    1005 |      35.00 |
|    1005 |      55.00 |
|    1002 |       8.99 |
|    1003 |      50.00 |
|    1003 |       4.49 |
+---------+------------+
12 rows in set (0.04 sec)
```

* LIMIT,限制输出结果

只返回5条结果

```sql
mysql> select prod_name from products limit 5;
+--------------+
| prod_name    |
+--------------+
| .5 ton anvil |
| 1 ton anvil  |
| 2 ton anvil  |
| Detonator    |
| Bird seed    |
+--------------+
5 rows in set (0.00 sec)
```

从行3开始（注意: 第一行是行0, 这里即相当与从第4行开始），返回5条结果

```sql
mysql> select prod_name from products limit 3, 5;

-- 上面的语句等价于
mysql> select prod_name from products limit 5, 3;
```


## 给查询结果排序

* ORDER BY, 默认是对结果进行升序（ASC 排序）， 如果需要指定降序则使用DESC


## 过滤数据

* msyql过滤时默认是不区分大小写的

```sql
mysql> select prod_name from products where prod_name = 'safe';
+-----------+
| prod_name |
+-----------+
| Safe      |
+-----------+
1 row in set (0.00 sec)
```

* BETWEEN

 查询价格在5～10美元之间的产品

```sql
 mysql> select prod_name, prod_price from products where prod_price between 5 and 10;
+----------------+------------+
| prod_name      | prod_price |
+----------------+------------+
| .5 ton anvil   |       5.99 |
| 1 ton anvil    |       9.99 |
| Bird seed      |      10.00 |
| Oil can        |       8.99 |
| TNT (5 sticks) |      10.00 |
+----------------+------------+
5 rows in set (0.00 sec)
```

* NULL, 无值，它与字段包含0, 空字符或仅仅包含空格**不同**

在查询某些不具有特定行的值的时候，你可能希望返回具有NULL的值，但是不行，因为未知具有特殊含义，数据库不知道他们是否匹配。

```sql
mysql> select cust_id, cust_email from customers;
+---------+---------------------+
| cust_id | cust_email          |
+---------+---------------------+
|   10001 | ylee@coyote.com     |
|   10002 | NULL                |
|   10003 | rabbit@wascally.com |
|   10004 | sam@yosemite.com    |
|   10005 | NULL                |
+---------+---------------------+
5 rows in set (0.00 sec)

mysql> select cust_id, cust_email from customers where cust_email != 'ylee@coyote.com';
+---------+---------------------+
| cust_id | cust_email          |
+---------+---------------------+
|   10003 | rabbit@wascally.com |
|   10004 | sam@yosemite.com    |
+---------+---------------------+
```

从上面的例子可以看出，NULL行并没有被返回。

因此，在过滤数据时，一定要验证返回的数据中确实给出了被过滤列具有NULL的行。



* 数据聚合

<pre>
SELECT [field1, field2, ...fieldn] fun_name
FROM tablename
[WHERE where_condition]
[GROUP BY field1, field2,...filedn
[WITH ROLLUP]
[HAVING where_condition]
</pre>

参数说明:
fun_name 表示要做的聚合操作，也就是聚合函数，常用的有sum(求和), count(\*)(记录数）, max， min。

GROUP BY 关键字表示要进行分类聚合的字段，比如按照部门分类统计员工数量，部门就
应该写在GROUP BY后面。

WITH ROLLUP是可选语法，表明是否对分类聚合后的结果进行再汇总

HAVING 关键字表示对分类后的结果在进行条件过滤。

**注意**:
having和where的区别在于，having是对聚合后的结果进行条件过滤，而where
是在聚合前就对记录进行过滤。如果逻辑允许，我们尽可能先过滤记录。

```sql
-- 统计总数
mysql> select count(1) from products;
+----------+
| count(1) |
+----------+
|       14 |
+----------+
1 row in set (0.01 sec)

-- 统计各个vend的总数
mysql> select vend_id, count(1) from products group by vend_id;
+---------+----------+
| vend_id | count(1) |
+---------+----------+
|    1001 |        3 |
|    1002 |        2 |
|    1003 |        7 |
|    1005 |        2 |
+---------+----------+
4 rows in set (0.01 sec)


-- 既要统计各个vend的总数, 又要统计所有的总数
mysql> select vend_id, count(1) from products group by vend_id with
rollup;
+---------+----------+
| vend_id | count(1) |
+---------+----------+
|    1001 |        3 |
|    1002 |        2 |
|    1003 |        7 |
|    1005 |        2 |
|    NULL |       14 |
+---------+----------+
5 rows in set (0.00 sec)


-- 既要统计各个vend的总数, 又要统计所有的总数, 并对条件做筛选。
-- 注意所有的总数仍然没有变，这是因为having是对聚合后的结果做筛选。
mysql> select vend_id, count(1) from products group by vend_id with rollup
having count(1)> 3;
+---------+----------+
| vend_id | count(1) |
+---------+----------+
|    1003 |        7 |
|    NULL |       14 |
+---------+----------+
2 rows in set (0.01 sec)

```


* 表连接

从大类上分，表连接分为内连接和外连接，她们之间最大的区别是: 内连接仅选出两张表
中相互匹配的记录，而外连接会选出其他不匹配的记录，我们最常用的是内连接。

外连接又分为左连接和右连接，具体定义如下。
左连接: 包含所有的左边表的记录甚至是右边表中没有和他匹配的记录。
右连接: 包含所有的右边表的记录设置是左边表中没有和他匹配的记录。

```sql

mysql> select * from vendors;
+---------+----------------+-----------------+-------------+------------+----------+--------------+
| vend_id | vend_name      | vend_address    | vend_city   | vend_state | vend_zip | vend_country |
+---------+----------------+-----------------+-------------+------------+----------+--------------+
|    1001 | Anvils R Us    | 123 Main Street | Southfield  | MI         | 48075    | USA          |
|    1002 | LT Supplies    | 500 Park Street | Anytown     | OH         | 44333    | USA          |
|    1003 | ACME           | 555 High Street | Los Angeles | CA         | 90046    | USA          |
|    1004 | Furball Inc.   | 1000 5th Avenue | New York    | NY         | 11111    | USA          |
|    1005 | Jet Set        | 42 Galaxy Road  | London      | NULL       | N16 6PS  | England      |
|    1006 | Jouets Et Ours | 1 Rue Amusement | Paris       | NULL       | 45678    | France       |
+---------+----------------+-----------------+-------------+------------+----------+--------------+
6 rows in set (0.00 sec)


mysql> select * from products;
+---------+---------+----------------+------------+----------------------------------------------------------------+
| prod_id | vend_id | prod_name      | prod_price | prod_desc                                                      |
+---------+---------+----------------+------------+----------------------------------------------------------------+
| ANV01   |    1001 | .5 ton anvil   |       5.99 | .5 ton anvil, black, complete with handy hook                  |
| ANV02   |    1001 | 1 ton anvil    |       9.99 | 1 ton anvil, black, complete with handy hook and carrying case |
| ANV03   |    1001 | 2 ton anvil    |      14.99 | 2 ton anvil, black, complete with handy hook and carrying case |
| DTNTR   |    1003 | Detonator      |      13.00 | Detonator (plunger powered), fuses not included                |
| FB      |    1003 | Bird seed      |      10.00 | Large bag (suitable for road runners)                          |
| FC      |    1003 | Carrots        |       2.50 | Carrots (rabbit hunting season only)                           |
| FU1     |    1002 | Fuses          |       3.42 | 1 dozen, extra long                                            |
| JP1000  |    1005 | JetPack 1000   |      35.00 | JetPack 1000, intended for single use                          |
| JP2000  |    1005 | JetPack 2000   |      55.00 | JetPack 2000, multi-use                                        |
| OL1     |    1002 | Oil can        |       8.99 | Oil can, red                                                   |
| SAFE    |    1003 | Safe           |      50.00 | Safe with combination lock                                     |
| SLING   |    1003 | Sling          |       4.49 | Sling, one size fits all                                       |
| TNT1    |    1003 | TNT (1 stick)  |       2.50 | TNT, red, single stick                                         |
| TNT2    |    1003 | TNT (5 sticks) |      10.00 | TNT, red, pack of 10 sticks                                    |
+---------+---------+----------------+------------+----------------------------------------------------------------+
14 rows in set (0.00 sec)


-- 内连接

mysql> select vendors.vend_id, vend_name, prod_name from vendors,  products
where vendors.vend_id = products.vend_id;
+---------+-------------+----------------+
| vend_id | vend_name   | prod_name      |
+---------+-------------+----------------+
|    1001 | Anvils R Us | .5 ton anvil   |
|    1001 | Anvils R Us | 1 ton anvil    |
|    1001 | Anvils R Us | 2 ton anvil    |
|    1002 | LT Supplies | Fuses          |
|    1002 | LT Supplies | Oil can        |
|    1003 | ACME        | Detonator      |
|    1003 | ACME        | Bird seed      |
|    1003 | ACME        | Carrots        |
|    1003 | ACME        | Safe           |
|    1003 | ACME        | Sling          |
|    1003 | ACME        | TNT (1 stick)  |
|    1003 | ACME        | TNT (5 sticks) |
|    1005 | Jet Set     | JetPack 1000   |
|    1005 | Jet Set     | JetPack 2000   |
+---------+-------------+----------------+
14 rows in set (0.00 sec)


--  左连接， 注意比上面的结果多了2条记录1004和1006的记录，因为products表中没有
-- 关于vend_id为1004和1006的记录

mysql> select vendors.vend_id, vend_name, prod_name from vendors left join
products on vendors.vend_id = products.vend_id;
+---------+----------------+----------------+
| vend_id | vend_name      | prod_name      |
+---------+----------------+----------------+
|    1001 | Anvils R Us    | .5 ton anvil   |
|    1001 | Anvils R Us    | 1 ton anvil    |
|    1001 | Anvils R Us    | 2 ton anvil    |
|    1002 | LT Supplies    | Fuses          |
|    1002 | LT Supplies    | Oil can        |
|    1003 | ACME           | Detonator      |
|    1003 | ACME           | Bird seed      |
|    1003 | ACME           | Carrots        |
|    1003 | ACME           | Safe           |
|    1003 | ACME           | Sling          |
|    1003 | ACME           | TNT (1 stick)  |
|    1003 | ACME           | TNT (5 sticks) |
|    1004 | Furball Inc.   | NULL           |
|    1005 | Jet Set        | JetPack 1000   |
|    1005 | Jet Set        | JetPack 2000   |
|    1006 | Jouets Et Ours | NULL           |
+---------+----------------+----------------+
16 rows in set (0.00 sec)


-- 右连接

mysql> select vendors.vend_id, vend_name, prod_name from vendors right join
products on vendors.vend_id = products.vend_id;
+---------+-------------+----------------+
| vend_id | vend_name   | prod_name      |
+---------+-------------+----------------+
|    1001 | Anvils R Us | .5 ton anvil   |
|    1001 | Anvils R Us | 1 ton anvil    |
|    1001 | Anvils R Us | 2 ton anvil    |
|    1003 | ACME        | Detonator      |
|    1003 | ACME        | Bird seed      |
|    1003 | ACME        | Carrots        |
|    1002 | LT Supplies | Fuses          |
|    1005 | Jet Set     | JetPack 1000   |
|    1005 | Jet Set     | JetPack 2000   |
|    1002 | LT Supplies | Oil can        |
|    1003 | ACME        | Safe           |
|    1003 | ACME        | Sling          |
|    1003 | ACME        | TNT (1 stick)  |
|    1003 | ACME        | TNT (5 sticks) |
+---------+-------------+----------------+
14 rows in set (0.00 sec)


-- 右连接和左连接类似，两者之间可以互相转化。
-- 区别在于FROM后面的表名

mysql> select vendors.vend_id, vend_name, prod_name from products left join
vendors on vendors.vend_id = products.vend_id;
+---------+-------------+----------------+
| vend_id | vend_name   | prod_name      |
+---------+-------------+----------------+
|    1001 | Anvils R Us | .5 ton anvil   |
|    1001 | Anvils R Us | 1 ton anvil    |
|    1001 | Anvils R Us | 2 ton anvil    |
|    1003 | ACME        | Detonator      |
|    1003 | ACME        | Bird seed      |
|    1003 | ACME        | Carrots        |
|    1002 | LT Supplies | Fuses          |
|    1005 | Jet Set     | JetPack 1000   |
|    1005 | Jet Set     | JetPack 2000   |
|    1002 | LT Supplies | Oil can        |
|    1003 | ACME        | Safe           |
|    1003 | ACME        | Sling          |
|    1003 | ACME        | TNT (1 stick)  |
|    1003 | ACME        | TNT (5 sticks) |
+---------+-------------+----------------+
14 rows in set (0.00 sec)

```

* 记录联合

我们经常会碰到这样的需求，将两个表的数据按照一定的查询条件查询出来之后，将结果
合并到一起显示出来。这个时候就需要用到 UNION 和 UNION ALL 关键字来实现这样的功能.

UNION 和 UNION ALL的主要区别在于UNION ALL是把所有的结果合并显示在一起，而
UNION是将UNION ALL之后的结果进行一次DISTINCT, 出去重复记录后的结果。
```sql
mysql> select * from vendors;
+---------+----------------+-----------------+-------------+------------+----------+--------------+
| vend_id | vend_name      | vend_address    | vend_city   | vend_state | vend_zip | vend_country |
+---------+----------------+-----------------+-------------+------------+----------+--------------+
|    1001 | Anvils R Us    | 123 Main Street | Southfield  | MI         | 48075    | USA          |
|    1002 | LT Supplies    | 500 Park Street | Anytown     | OH         | 44333    | USA          |
|    1003 | ACME           | 555 High Street | Los Angeles | CA         | 90046    | USA          |
|    1004 | Furball Inc.   | 1000 5th Avenue | New York    | NY         | 11111    | USA          |
|    1005 | Jet Set        | 42 Galaxy Road  | London      | NULL       | N16 6PS  | England      |
|    1006 | Jouets Et Ours | 1 Rue Amusement | Paris       | NULL       | 45678    | France       |
+---------+----------------+-----------------+-------------+------------+----------+--------------+
6 rows in set (0.08 sec)

mysql> select * from products;
+---------+---------+----------------+------------+----------------------------------------------------------------+
| prod_id | vend_id | prod_name      | prod_price | prod_desc                                                      |
+---------+---------+----------------+------------+----------------------------------------------------------------+
| ANV01   |    1001 | .5 ton anvil   |       5.99 | .5 ton anvil, black, complete with handy hook                  |
| ANV02   |    1001 | 1 ton anvil    |       9.99 | 1 ton anvil, black, complete with handy hook and carrying case |
| ANV03   |    1001 | 2 ton anvil    |      14.99 | 2 ton anvil, black, complete with handy hook and carrying case |
| DTNTR   |    1003 | Detonator      |      13.00 | Detonator (plunger powered), fuses not included                |
| FB      |    1003 | Bird seed      |      10.00 | Large bag (suitable for road runners)                          |
| FC      |    1003 | Carrots        |       2.50 | Carrots (rabbit hunting season only)                           |
| FU1     |    1002 | Fuses          |       3.42 | 1 dozen, extra long                                            |
| JP1000  |    1005 | JetPack 1000   |      35.00 | JetPack 1000, intended for single use                          |
| JP2000  |    1005 | JetPack 2000   |      55.00 | JetPack 2000, multi-use                                        |
| OL1     |    1002 | Oil can        |       8.99 | Oil can, red                                                   |
| SAFE    |    1003 | Safe           |      50.00 | Safe with combination lock                                     |
| SLING   |    1003 | Sling          |       4.49 | Sling, one size fits all                                       |
| TNT1    |    1003 | TNT (1 stick)  |       2.50 | TNT, red, single stick                                         |
| TNT2    |    1003 | TNT (5 sticks) |      10.00 | TNT, red, pack of 10 sticks                                    |
+---------+---------+----------------+------------+----------------------------------------------------------------+
14 rows in set (0.00 sec)

mysql> select vend_id from vendors union all select vend_id from products;
+---------+
| vend_id |
+---------+
|    1001 |
|    1002 |
|    1003 |
|    1004 |
|    1005 |
|    1006 |
|    1001 |
|    1001 |
|    1001 |
|    1002 |
|    1002 |
|    1003 |
|    1003 |
|    1003 |
|    1003 |
|    1003 |
|    1003 |
|    1003 |
|    1005 |
|    1005 |
+---------+
20 rows in set (0.12 sec)

mysql> select vend_id from vendors union select vend_id from products;
+---------+
| vend_id |
+---------+
|    1001 |
|    1002 |
|    1003 |
|    1004 |
|    1005 |
|    1006 |
+---------+
6 rows in set (0.01 sec)

```

* 同配符
% : 表示任何字符出现任意次数
_ : 表示匹配单个字符

```sql
mysql> select prod_id, prod_name from products where prod_name LIKE '_ ton anvil';
+---------+-------------+
| prod_id | prod_name   |
+---------+-------------+
| ANV02   | 1 ton anvil |
| ANV03   | 2 ton anvil |
+---------+-------------+
2 rows in set (0.00 sec)
```

* 正则表达式

使用REGEXP关键字
```sql
mysql> select prod_name from products where prod_name REGEXP '1000' order by prod_name;
+--------------+
| prod_name    |
+--------------+
| JetPack 1000 |
+--------------+
1 row in set (0.25 sec)


-- 需要注意的是: 在mysql中，转移需要使用'\\'。
mysql> select vend_name from vendors where vend_name REGEXP '\\.';
+--------------+
| vend_name    |
+--------------+
| Furball Inc. |
+--------------+
1 row in set (0.03 sec)


-- 简单的正则表达式测试
-- 可以在不使用数据库表的情况下使用SELECT来测试正则表达式，REGEXP检查总是返回0（没有匹配）或1（匹配）。
mysql> select 'hello' REGEXP '[0-9]';
+------------------------+
| 'hello' REGEXP '[0-9]' |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.00 sec)
```

* 字符串拼接

MYSQL使用Concat()函数来实现，而大多数其他的DBMS使用 + 来实现
```sql
mysql> help Concat
Name: 'CONCAT'
Description:
Syntax:
CONCAT(str1,str2,...)

Returns the string that results from concatenating the arguments. May
have one or more arguments. If all arguments are nonbinary strings, the
result is a nonbinary string. If the arguments include any binary
strings, the result is a binary string. A numeric argument is converted
to its equivalent string form. This is a nonbinary string as of MySQL
5.5.3. Before 5.5.3, it is a binary string; to to avoid that and
produce a nonbinary string, you can use an explicit type cast, as in
this example:

SELECT CONCAT(CAST(int_col AS CHAR), char_col);

CONCAT() returns NULL if any argument is NULL.

URL: http://dev.mysql.com/doc/refman/5.5/en/string-functions.html

Examples:
mysql> SELECT CONCAT('My', 'S', 'QL');
        -> 'MySQL'
mysql> SELECT CONCAT('My', NULL, 'QL');
        -> NULL
mysql> SELECT CONCAT(14.3);
        -> '14.3'
```

## MySQL函数

大多数的SQL支持一下类型的函数

* 用于处理文本（如删除或填充，转换值为大写或小写）的文本函数
* 用于在数值数据上进行算术操作（如返回绝对值，进行代数运算）
* 用于处理日期和时间并从这些值中提取特定成分(例如，返回两个日期之差，检查日期有效性）的日期和时间函数
* 返回DBMS正在使用的特殊信息（如返回用户登录信息，检查版本细节）的系统函数

常见的文本处理函数

|  函数       |      说明        |
| ----------- | ---------------- |
| Left()      | 返回串左边的函数 |
| Right()     | 返回串右边的字符 |
| Length()    | 返回串的长度     |
| Locate()    | 找出串的一个子串 |
| Lower()     | 将串返回小写     |
| Upper()     | 将串转换为大写   |
| LTrim()     | 去掉串左边的空格 |
| RTrim()     | 去掉串右边的空格 |
| Soundex()   | 返回串的SOUNDEX值|
| SubString() | 返回子串的字符   |

具体使用自行查看帮助`help 函数名`

SOUNDEX 是一个将任何文本串转换为描述其语音表示的字母数字模式的算法，SOUNDEX考虑了类似的发音字符和音节，
使得能对串进行发音比较而不是字母比较。


## MySQL支持的数据类型

### 数值类型

<table>
    <caption>MySQL中的数值类型</caption>
    <tr>
        <th>整数类型</th>
        <th>字节</th>
        <th>最小值</th>
        <th>最大值</th>
    </tr>
    <tr>
        <td>TINYINT</td>
        <td>1</td>
        <td >有符号 -128<br />无符号 0</td>
        <td>有符号 127<br />无符号 255</td>
    </tr>
    <tr>
        <td>SMALLINT</td>
        <td>2</td>
        <td>有符号 -32768<br />无符号 0</td>
        <td>有符号 32767<br />无符号 65535</td>
    </tr>
    <tr>
        <td>MEDIUMINT</td>
        <td>3</td>
        <td>有符号 -8388608<br />无符号 0</td>
        <td>有符号 8388607<br />无符号 1677215</td>
    </tr>
    <tr>
        <td>INT、INTEGER</td>
        <td>4</td>
        <td>有符号 -2147483648<br />无符号 0</td>
        <td>有符号 2147483647<br />无符号 4294967295</td>
    </tr>
    <tr>
        <td>BIGINT</td>
        <td>8</td>
        <td>有符号 -9223372036854775808<br />无符号 0</td>
        <td>有符号 9223372036854775807<br />无符号 18446744073709551615</td>
    </tr>
    <tr>
        <th>浮点类型</th>
        <th>字节</th>
        <th>最小值</th>
        <th>最大值</th>
    </tr>
    <tr>
        <td>FLOAT</td>
        <td>4</td>
        <td>---</td>
        <td>---</td>
    </tr>
    <tr>
        <td>DOUBLE</td>
        <td>8</td>
        <td>---</td>
        <td>---</td>
    </tr>
    <tr>
        <th>定点数类型</th>
        <th>字节</th>
        <th colspan="2">描述</th>
    </tr>
    <tr>
        <td>DEC(M, D)</td>
        <td rowspan="2">M+2</td>
        <td colspan="2" rowspan="2">最大取值范围与DOUBLE相同，给定DECIMAL的有效取值范围由M和D决定</td>
    </tr>
    <tr>
        <td>DECIMAL(M,D)</td>
    </tr>
    <tr>
        <th>位类型</th>
        <th>字节</th>
        <th>最小值</th>
        <th>最大值</th>
    </tr>
    <tr>
        <td>BIT</td>
        <td>1～8</td>
        <td>BIT(1)</td>
        <td>BIT(8)</td>
    </tr>
</table>


对于整数类型，MySQL还支持在类型名称后面的小括号制定显示宽度，例如int(5)表示宽度小于5位的时候在前面填充宽度，如果不指定则默认宽度为int(11).
当插入的数据超过了设置宽度时，仍然可以插入数据，只是宽度格式就没有任何意义了。

```sql
mysql> create table t1(id1 int, id2 int(5));
Query OK, 0 rows affected (0.11 sec)

mysql> insert into t1 values(1, 1);
Query OK, 1 row affected (0.06 sec)

mysql> select * from t1;
+------+------+
| id1  | id2  |
+------+------+
|    1 |    1 |
+------+------+
1 row in set (0.00 sec)

mysql> desc t1;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id1   | int(11) | YES  |     | NULL    |       |
| id2   | int(5)  | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
2 rows in set (0.01 sec)

mysql> alter table t1 modify id1 int zerofill;
Query OK, 1 row affected (0.30 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> desc t1;
+-------+---------------------------+------+-----+---------+-------+
| Field | Type                      | Null | Key | Default | Extra |
+-------+---------------------------+------+-----+---------+-------+
| id1   | int(10) unsigned zerofill | YES  |     | NULL    |       |
| id2   | int(5)                    | YES  |     | NULL    |       |
+-------+---------------------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> select * from t1;
+------------+------+
| id1        | id2  |
+------------+------+
| 0000000001 |    1 |
+------------+------+
1 row in set (0.00 sec)
```

对于小数的表示，MySQL分为两种方式: 浮点数和定点数。 浮点数包括float(单精度)和double(双精度), 而定点数则只有一种表示。
定点数在MySQL内部以字符串的形式存放，比浮点数更精确， 适合用来存放货币等精度高的数据。

浮点数和定点数都可以用类型名称后面加"(M, D)"的方式来表示，“(M, D)"表示该值一共显示M位数字（整数位 + 小数位),其中D位于小数点后面， M和D又称为**精度**和**标度**。

float和double在不指定精度时，默认会按照实际的精度(由实际的硬件和操作系统来决定)来显示，而decimal在不指定精度的时候，默认的整数数位为10, 默认的小数数位为0.

```sql
-- 创建表
mysql> create table t2(id1 float(5,2), id2 double(5, 2), id3 decimal(5,2));
Query OK, 0 rows affected (0.12 sec)

mysql> desc t2;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id1   | float(5,2)   | YES  |     | NULL    |       |
| id2   | double(5,2)  | YES  |     | NULL    |       |
| id3   | decimal(5,2) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

-- 插入数据1.23， 没有异常
mysql> insert into t2 values(1.23, 1.23, 1.23);
Query OK, 1 row affected (0.04 sec)

mysql> select * from t2;
+------+------+------+
| id1  | id2  | id3  |
+------+------+------+
| 1.23 | 1.23 | 1.23 |
+------+------+------+
1 row in set (0.02 sec)

-- 向id1, id2 插入数据1.234， id3插入1.23
mysql> insert into t2 values(1.234, 1.234, 1.23);
Query OK, 1 row affected (0.04 sec)

-- 可以看到id1, id2 由于精度限制，最后一位四舍五入了
mysql> select * from t2;
+------+------+------+
| id1  | id2  | id3  |
+------+------+------+
| 1.23 | 1.23 | 1.23 |
| 1.23 | 1.23 | 1.23 |
+------+------+------+
2 rows in set (0.00 sec)

-- 都插入数据1.23， 出现警告
mysql> insert into t2 values(1.234, 1.234, 1.234);
Query OK, 1 row affected, 1 warning (0.08 sec)

-- id3由于精度限制被截断了
mysql> show warnings;
+-------+------+------------------------------------------+
| Level | Code | Message                                  |
+-------+------+------------------------------------------+
| Note  | 1265 | Data truncated for column 'id3' at row 1 |
+-------+------+------------------------------------------+
1 row in set (0.01 sec)

mysql> select * from t2;
+------+------+------+
| id1  | id2  | id3  |
+------+------+------+
| 1.23 | 1.23 | 1.23 |
| 1.23 | 1.23 | 1.23 |
| 1.23 | 1.23 | 1.23 |
+------+------+------+
3 rows in set (0.00 sec)

-- 去掉id1, id2, id3的精度限制
mysql> alter table t2 modify id1 float;
Query OK, 0 rows affected (0.08 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table t2 modify id2 float;
Query OK, 3 rows affected (0.24 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> alter table t2 modify id3 decimal;
Query OK, 3 rows affected, 3 warnings (0.26 sec)
Records: 3  Duplicates: 0  Warnings: 3

-- 可以看到decimal的默认精度限制为(10, 0)
mysql> desc t2;
+-------+---------------+------+-----+---------+-------+
| Field | Type          | Null | Key | Default | Extra |
+-------+---------------+------+-----+---------+-------+
| id1   | float         | YES  |     | NULL    |       |
| id2   | float         | YES  |     | NULL    |       |
| id3   | decimal(10,0) | YES  |     | NULL    |       |
+-------+---------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> insert into t2 values(1.234, 1.234, 1.234);
Query OK, 1 row affected, 1 warning (0.07 sec)

mysql> show warnings ;
+-------+------+------------------------------------------+
| Level | Code | Message                                  |
+-------+------+------------------------------------------+
| Note  | 1265 | Data truncated for column 'id3' at row 1 |
+-------+------+------------------------------------------+
1 row in set (0.00 sec)

mysql> select * from t2;
+-------+-------+------+
| id1   | id2   | id3  |
+-------+-------+------+
|  1.23 |  1.23 |    1 |
|  1.23 |  1.23 |    1 |
|  1.23 |  1.23 |    1 |
| 1.234 | 1.234 |    1 |
+-------+-------+------+
4 rows in set (0.00 sec)

```

对于BIT(位）类型，用于存放位字段值， BIT(M)可以用来存放多位二进制数，
M范围从1～64, 如果不写默认为1位，对于位字段，直接使用
SELECT命令将看不到结果，可以用bin()(显示为二进制格式）或者hex()（显示为十六进制格式）函数进行读取。
```sql
mysql> create table te (id BIT);
Query OK, 0 rows affected (0.12 sec)

-- 插入的BIT类型默认为1位

mysql> desc te;
+-------+--------+------+-----+---------+-------+
| Field | Type   | Null | Key | Default | Extra |
+-------+--------+------+-----+---------+-------+
| id    | bit(1) | YES  |     | NULL    |       |
+-------+--------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> insert into te values(1);
Query OK, 1 row affected (0.05 sec)

-- 直接用select是看不到结果的
mysql> select * from te;
+------+
| id   |
+------+
|      |
+------+
1 row in set (0.02 sec)

mysql> select bin(id), hex(id) from te;
+---------+---------+
| bin(id) | hex(id) |
+---------+---------+
| 1       | 1       |
+---------+---------+
1 row in set (0.00 sec)

-- 数据插入bit类型字段时，首先转换为二进制，如果位数允许，将成功插入。
-- 2的二进制码是"10", 超过位数1, 插入失败。
mysql> insert into te values(2);
Query OK, 1 row affected, 1 warning (0.04 sec)

mysql> show warnings ;
+---------+------+---------------------------------------------+
| Level   | Code | Message                                     |
+---------+------+---------------------------------------------+
| Warning | 1264 | Out of range value for column 'id' at row 1 |
+---------+------+---------------------------------------------+
1 row in set (0.00 sec)

mysql> select bin(id), hex(id) from te;
+---------+---------+
| bin(id) | hex(id) |
+---------+---------+
| 1       | 1       |
| 1       | 1       |
+---------+---------+
2 rows in set (0.00 sec)

mysql>

```

### 日期时间类型

* 如果要用来表示年月日，通常用DATE表示
* 如果要用来表示年月日时分秒，通常用DATETIME表示
* 如果要用来表示时分秒，通常用TIME表示
* 如果需要经常插入或者更新为当前系统时间，通常用TIMESTAMP来表示
* 如果只表示年份，可以用YEAR, 它比DATE占用更少的时间

```sql
mysql> create table t(d date, t time, dt datetime);
Query OK, 0 rows affected (0.17 sec)

mysql> desc t;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| d     | date     | YES  |     | NULL    |       |
| t     | time     | YES  |     | NULL    |       |
| dt    | datetime | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> insert into t values(now(), now(), now());
Query OK, 1 row affected, 1 warning (0.09 sec)

mysql> select * from t;
+------------+----------+---------------------+
| d          | t        | dt                  |
+------------+----------+---------------------+
| 2014-01-23 | 11:32:18 | 2014-01-23 11:32:18 |
+------------+----------+---------------------+
1 row in set (0.01 sec)

```
