---
title: MySQL8.0官方文档学习笔记
date: 2020-10-30 18:03:14
tags: [MySQL]
---

https://dev.mysql.com/doc/refman/8.0/en/introduction.html

# 第1章 MySQL概要

MySQL提供了一个非常快, 多线程, 多用户, 健壮的SQL数据库服务。

MySQL提供了两个版本，一个是GNU协议开源，一个是商业版。

## 1.1 关于该手册

​	基于MySQL8.0, 版本为8.0.24。可能不适用于以前版本。

​	本手册仅供参考，因此不提供有关SQL或关系数据库概念的一般说明。它也不会教你如何使用你的操作系统或命令行解释器。

### 排版与语法约定

略

## 1.2 MySQL 数据管理系统概要

### 1.2.1 什么是MySQL？

​	MySQL是一个Oracle公司开发，发布和支持的一个非常流行的开源SQL数据库管理系统。

* MySQL是一个数据库管理系统

* MySQL是一个关系型数据库系统。

  关系型数据库将数据存储在单独的表中，而不是将所有的数据放到一个大的存储空间中。

* MySQL是开源的。

* MySQL非常快, 可靠, 可扩展, 易用。

* MySql适用于客户端、服务端以及嵌入式系统。

* 有大量关于MySQL的软件。

### 1.2.2 MySQL的主要特性

​	如果想看各个版本的不同可以参考如下手册：

- MySQL 8.0: [Section 1.3, “What Is New in MySQL 8.0”](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)
- MySQL 5.7: [What Is New in MySQL 5.7](https://dev.mysql.com/doc/refman/5.7/en/mysql-nutshell.html)
- MySQL 5.6: [What Is New in MySQL 5.6](https://dev.mysql.com/doc/refman/5.6/en/mysql-nutshell.html)

#### 内部结构与可移植性(Internals and Portability)

* 使用C和C++编写。
* 使用不同编译器进行了广泛地测试。
* 可以在不同平台工作https://www.mysql.com/support/supportedplatforms/database.html
* 为了方便移植，使用**CMAKE**配置。
* 使用Purify和Valgrind测试。
* 使用独立模块多层服务设计。
* 设计为完全多线程使用内核线程，以便轻松使用多个CPU（如果可用)。
* 提供了事务型和非事务型存储引擎。
* 使用了非常快速的B树磁盘表(MyISAM)和索引压缩
* 添加其他存储引擎相当容易。如果你想为一个内部数据库提供SQL接口，这个特性非常有用。
* 使用非常快速的基于线程的内存分配系统。
* 使用优化的嵌套循环jion执行非常快速的join。
* 实现了内存hash表，用作临时表。
* 使用高度优化的类库实现SQL函数，该类库应尽可能快。通常在查询初始化之后根本没有内存分配。
* 将服务器作为单独的程序提供，以在客户机/服务器网络环境中使用，并作为可嵌入（链接）到独立应用程序中的库。这样的应用程序可以单独使用，也可以在没有可用网络的环境中使用。

#### 数据类型

- Many data types: signed/unsigned integers 1, 2, 3, 4, and 8 bytes long, [`FLOAT`](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html), [`DOUBLE`](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html), [`CHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html), [`VARCHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html), [`BINARY`](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html), [`VARBINARY`](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html), [`TEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html), [`BLOB`](https://dev.mysql.com/doc/refman/8.0/en/blob.html), [`DATE`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html), [`TIME`](https://dev.mysql.com/doc/refman/8.0/en/time.html), [`DATETIME`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html), [`TIMESTAMP`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html), [`YEAR`](https://dev.mysql.com/doc/refman/8.0/en/year.html), [`SET`](https://dev.mysql.com/doc/refman/8.0/en/set.html), [`ENUM`](https://dev.mysql.com/doc/refman/8.0/en/enum.html), and OpenGIS spatial types. See [Chapter 11, *Data Types*](https://dev.mysql.com/doc/refman/8.0/en/data-types.html).
- Fixed-length and variable-length string types.

#### 语句与函数

* 在SELECT和WHERE子句查询中支持所有操作符和函数。
* 支持GROUP BY 和 ORDER BY子句。 支持group 函数[`COUNT()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_count), [`AVG()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_avg), [`STD()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_std), [`SUM()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_sum), [`MAX()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_max), [`MIN()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_min), and [`GROUP_CONCAT()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_group-concat)
* 支持表和字段的别名。
* 支持`DELETE`, `INSERT`,`REPLACE`,`UPDATE`返回受影响的条数，或者通过在连接到服务器时设置标志来返回匹配的行数。
* 支持MySQL独有的SHOW语句来显示databases, storage engines, tables, and indexes的信息。支持`INFORMATION_SCHEMA`数据库。
* 支持`EXPLAIN`语句来展示优化器如何解析查询。
* 表、列以及函数名都是独立存在的，比如,`ABS`也是一个合法的列名。
* 可以在同一语句中引用来自不同数据库的表。

#### 安全性

* 权限和密码系统非常灵活和安全，并且支持主机验证。
* 当连接到服务器时，通过加密所有密码流量来实现密码安全。

#### 可扩展性和限制

* 支持很大的数据库。我们使用MySQL数据库包含5000万条记录。并且知道用户可以有20万张表和50亿行数据。
* 每张表最多支持64个索引。每个索引可以包含1到16列或部分列组成。InnoDB支持的最大索引宽度是767字节或者3072字节，See [Section 15.22, “InnoDB Limits”](https://dev.mysql.com/doc/refman/8.0/en/innodb-limits.html)。MyISAM的最大索引宽度是1000字节。See [Section 16.2, “The MyISAM Storage Engine”](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html).CHAR, VARCHAR, BLOB和TEXT列支持前缀索引。

#### Connectivity

* Client可以使用几种协议连接MySQL。

  * 任何平台上的TCP/IP sockets。
  * Windows系统中，如果服务器在启用命名的“name_pipe”系统变量的情况下启动, 可以使用named_pipes连接。Windows server也支持共享内存连接如果以`shared_memory`系统变量启用。Client可以使用 [`--protocol=memory`](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_protocol)通过共享内存连接。
  * 在Unix系统中，client可以使用Unix domain socket files连接。

* MySQL client程序由很多语言写成。A client library written in C is available for clients written in C or C++, or for any language that provides C bindings.
* C, C++, Eiffel, Java, Perl, PHP, Python, Ruby, and Tcl 都可以使用API。
* Connector/ODBC（MyODBC）接口为使用ODBC（开放数据库连接）连接的客户端程序提供MySQL支持。
* Connector/J接口为使用JDBC连接的Java客户机程序提供MySQL支持。
* MySQL连接器/NET使开发人员能够轻松地创建需要与MySQL进行安全、高性能数据连接的.NET应用程序。

#### 本地化

* The server can provide error messages to clients in many languages. See [Section 10.12, “Setting the Error Message Language”](https://dev.mysql.com/doc/refman/8.0/en/error-message-language.html).
* 支持集中不同字符集的全支持，包括`latin1` (cp1252), `german`, `big5`, `ujis`, several Unicode character sets, and more。
* 所有数据都按选择的数据集保存。
* 根据默认字符集和排序规则进行排序和比较，可以在MySQL启动时调整。see [Section 10.3.2, “Server Character Set and Collation”](https://dev.mysql.com/doc/refman/8.0/en/charset-server.html))。
* 服务器时区可以动态更改，单个客户端可以指定自己的时区。See [Section 5.1.15, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)。

#### Client And Tools

- MySQL includes several client and utility programs. These include both command-line programs such as [**mysqldump**](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) and [**mysqladmin**](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html), and graphical programs such as [MySQL Workbench](https://dev.mysql.com/doc/refman/8.0/en/workbench.html).
- MySQL Server has built-in support for SQL statements to check, optimize, and repair tables. These statements are available from the command line through the [**mysqlcheck**](https://dev.mysql.com/doc/refman/8.0/en/mysqlcheck.html) client. MySQL also includes [**myisamchk**](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html), a very fast command-line utility for performing these operations on `MyISAM` tables. See [Chapter 4, *MySQL Programs*](https://dev.mysql.com/doc/refman/8.0/en/programs.html).
- MySQL programs can be invoked with the `--help` or `-?` option to obtain online assistance.

### 1.2.3 MySQL的历史

略

## 1.3 MySQL8.0新特性

//todo 

https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html

## 1.4 Server and Status Variables and Options Added, Deprecated, or Removed in MySQL 8.0

 //todo

https://dev.mysql.com/doc/refman/8.0/en/added-deprecated-removed.html

## 1.5 MySQL Information Sources

  https://dev.mysql.com/doc/refman/8.0/en/information-sources.html

## 1.6 How to Report Bugs or Problems

https://dev.mysql.com/doc/refman/8.0/en/bug-reports.html

## 1.7 MySQL Standards Compliance

​	We are not afraid to add extensions to SQLor support for non-SQL features if this greatly increases the usability of MySQL Server for a largesegment of our user base. 

​	MySQL supports ODBC levels 0 to 3.51.

​	MySQL supports high-availability database clustering using the NDBCLUSTER storage engine. See Chapter 23, MySQL NDB Cluster 8.0.

​	We implement XML functionality which supports most of the W3C XPath standard. See Section 12.12, “XML Functions”.

​	MySQL supports a native JSON data type as defined by RFC 7159, and based on the ECMAScript standard (ECMA-262). See Section 11.5, “The JSON Data Type”. MySQL also implements a subset of the SQL/JSON functions specified by a pre-publication draft of the SQL:2016 standard; see Section 12.18, “JSON Functions”, for more information.

**Selecting SQL Modes**

​	The MySQL server can operate in different SQL modes, and can apply these modes differently for different clients, depending on the value of the sql_mode system variable. DBAs can set the global SQL mode to match site server operating requirements, and each application can set its session SQL mode to its own requirements.

​	Modes affect the SQL syntax MySQL supports and the data validation checks it performs. This makes it easier to use MySQL in different environments and to use MySQL together with other database servers.

​	For more information on setting the SQL mode, see Section 5.1.11, “Server SQL Modes”.

**Running MySQL in ANSI Mode**

​	To run MySQL Server in ANSI mode, start mysqld with the `--ansi` option. Running the server in ANSI mode is the same as starting it with the following options:

```mysql
--transaction-isolation=SERIALIZABLE --sql-mode=ANSI
```

​	To achieve the same effect at runtime, execute these two statements:··

```mysql
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET GLOBAL sql_mode = 'ANSI';
```

​	You can see that setting the sql_mode system variable to 'ANSI' enables all SQL mode options that are relevant for ANSI mode as follows:

```mysql
mysql> SET GLOBAL sql_mode='ANSI';
mysql> SELECT @@GLOBAL.sql_mode;
-> 'REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI'
```

​	Running the server in ANSI mode with --ansi is not quite the same as setting the SQL mode to 'ANSI' because the --ansi option also sets the transaction isolation level.

​	See Section 5.1.7, “Server Command Options”.

### **1.7.1 MySQL Extensions to Standard SQL**

​	MySQL Server supports some extensions that you are not likely to find in other SQL DBMSs. Be warned that if you use them, your code is most likely not portable to other SQL servers. In some cases, you can write code that includes MySQL extensions, but is still portable, by using comments of the following form:

```mysql
/*! MySQL-specific code */
```

​	If you add a version number after the ! character, the syntax within the comment is executed only if the MySQL version is greater than or equal to the specified version number. The KEY_BLOCK_SIZE clause in the following comment is executed only by servers from MySQL 5.1.10 or higher:

```mysql
CREATE TABLE t1(a INT, KEY (a)) /*!50110 KEY_BLOCK_SIZE=1024 */;
```

The following descriptions list MySQL extensions, organized by category.

* Organization of data on disk
  * MySQL Server maps each database to a directory under the MySQL data directory, and maps tables within a database to file names in the database directory. Consequently, database and table names are case-sensitive in MySQL Server on operating systems that have case-sensitive file names (such as most Unix systems). See Section 9.2.3, “Identifier Case Sensitivity”.
* General language syntax
  * By default, strings can be enclosed by " as well as '. If the ANSI_QUOTES SQL mode is enabled, strings can be enclosed only by ' and the server interprets strings enclosed by " as identifiers.
  * \ is the escape character in strings.
  * In SQL statements, you can access tables from different databases with the db_name.tbl_name syntax. Some SQL servers provide the same functionality but call this User space. MySQL Server doesn't support tablespaces such as used in statements like this: CREATE TABLE ralph.my_table ... IN my_tablespace.
* SQL statement syntax
  * The ANALYZE TABLE, CHECK TABLE, OPTIMIZE TABLE, and REPAIR TABLE statements.
  * The CREATE DATABASE, DROP DATABASE, and ALTER DATABASE statements. See Section 13.1.12, “CREATE DATABASE Statement”, Section 13.1.24, “DROP DATABASE Statement”, and Section 13.1.2, “ALTER DATABASE Statement”.
  * The DO statement.
  * EXPLAIN SELECT to obtain a description of how tables are processed by the query optimizer.
  * The FLUSH and RESET statements.
  * The SET statement. See Section 13.7.6.1, “SET Syntax for Variable Assignment”.
  * The SHOW statement. See Section 13.7.7, “SHOW Statements”. The information produced by many of the MySQL-specific SHOW statements can be obtained in more standard fashion by  using SELECT to query INFORMATION_SCHEMA. See Chapter 26, INFORMATION_SCHEMA Tables
  * Use of LOAD DATA. In many cases, this syntax is compatible with Oracle LOAD DATA. See
    Section 13.2.7, “LOAD DATA Statement”.
  * Use of RENAME TABLE. See Section 13.1.36, “RENAME TABLE Statement”.
  * Use of REPLACE instead of DELETE plus INSERT. See Section 13.2.9, “REPLACE Statement”.
  * Use of CHANGE col_name, DROP col_name, or DROP INDEX, IGNORE or RENAME in ALTER
    TABLE statements. Use of multiple ADD, ALTER, DROP, or CHANGE clauses in an ALTER TABLE
    statement. See Section 13.1.9, “ALTER TABLE Statement”.
  * Use of index names, indexes on a prefix of a column, and use of INDEX or KEY in CREATE TABLE
    statements. See Section 13.1.20, “CREATE TABLE Statement”.
  * Use of TEMPORARY or IF NOT EXISTS with CREATE TABLE.
  * Use of IF EXISTS with DROP TABLE and DROP DATABASE.
  * The capability of dropping multiple tables with a single DROP TABLE statement
  * The ORDER BY and LIMIT clauses of the UPDATE and DELETE statements.
  * INSERT INTO tbl_name SET col_name = ... syntax.
  * The DELAYED clause of the INSERT and REPLACE statements.
  * The LOW_PRIORITY clause of the INSERT, REPLACE, DELETE, and UPDATE statements.
  * Use of INTO OUTFILE or INTO DUMPFILE in SELECT statements. See Section 13.2.10,
    “SELECT Statement”.
  * Options such as STRAIGHT_JOIN or SQL_SMALL_RESULT in SELECT statements.
  * You don't need to name all selected columns in the GROUP BY clause. This gives better
    performance for some very specific, but quite normal queries. See Section 12.20, “Aggregate
    Functions”.
  * You can specify ASC and DESC with GROUP BY, not just with ORDER BY.
  * The ability to set variables in a statement with the := assignment operator. See Section 9.4, “User-
    Defined Variables”.
* Data types
  * The MEDIUMINT, SET, and ENUM data types, and the various BLOB and TEXT data types.
  * The AUTO_INCREMENT, BINARY, NULL, UNSIGNED, and ZEROFILL data type attributes.

* Functions and operators
  * To make it easier for users who migrate from other SQL environments, MySQL Server supports aliases for many functions. For example, all string functions support both standard SQL syntax  and ODBC syntax.
  * MySQL Server understands the || and && operators to mean logical OR and AND, as in the C programming language. In MySQL Server, || and OR are synonyms, as are && and AND. Because of this nice syntax, MySQL Server doesn't support the standard SQL || operator for string concatenation; use CONCAT() instead. Because CONCAT() takes any number of arguments, it is easy to convert use of the || operator to MySQL Server.
  * Use of COUNT(DISTINCT value_list) where value_list has more than one element.
  * String comparisons are case-insensitive by default, with sort ordering determined by the collation of the current character set, which is utf8mb4 by default. To perform case-sensitive comparisons instead, you should declare your columns with the BINARY attribute or use the BINARY cast,  which causes comparisons to be done using the underlying character code values rather than a lexical ordering.
  * The % operator is a synonym for MOD(). That is, N % M is equivalent to MOD(N,M). % is supported for C programmers and for compatibility with PostgreSQL.
  * The =, <>, <=, <, >=, >, <<, >>, <=>, AND, OR, or LIKE operators may be used in expressions in the output column list (to the left of the FROM) in SELECT statements. For example:

```mysql
mysql> SELECT col1=1 AND col2=2 FROM my_table;
```

* 
  * The LAST_INSERT_ID() function returns the most recent AUTO_INCREMENT value. See
    Section 12.16, “Information Functions”.
  * LIKE is permitted on numeric values.
  * The REGEXP and NOT REGEXP extended regular expression operators.
  * CONCAT() or CHAR() with one argument or more than two arguments. (In MySQL Server, these
    functions can take a variable number of arguments.)
  * The BIT_COUNT(), CASE, ELT(), FROM_DAYS(), FORMAT(), IF(), MD5(), PERIOD_ADD(),
    PERIOD_DIFF(), TO_DAYS(), and WEEKDAY() functions.
  * Use of TRIM() to trim substrings. Standard SQL supports removal of single characters only.
  * The GROUP BY functions STD(), BIT_OR(), BIT_AND(), BIT_XOR(), and GROUP_CONCAT().
    See Section 12.20, “Aggregate Functions”.

### 1.7.2 MySQL Differences from Standard SQL

​	We try to make MySQL Server follow the ANSI SQL standard and the ODBC SQL standard, but MySQL Server performs operations differently in some cases:

* There are several differences between the MySQL and standard SQL privilege systems. For example, in MySQL, privileges for a table are not automatically revoked when you delete a table. You must explicitly issue a REVOKE statement to revoke privileges for a table. For more information, see Section 13.7.1.8, “REVOKE Statement”.
* The CAST() function does not support cast to REAL or BIGINT. See Section 12.11, “Cast Functions and Operators”.

#### 1.7.2.1 SELECT INTO TABLE Differences

​	MySQL Server doesn't support the SELECT ... INTO TABLE Sybase SQL extension. Instead, MySQL Server supports the INSERT INTO ... SELECT standard SQL syntax, which is basically the same thing. See Section 13.2.6.1, “INSERT ... SELECT Statement”. For example:

```mysql
INSERT INTO tbl_temp2 (fld_id)
SELECT tbl_temp1.fld_order_id
FROM tbl_temp1 WHERE tbl_temp1.fld_order_id > 100;
```

​	Alternatively, you can use SELECT ... INTO OUTFILE or CREATE TABLE ... SELECT.

​	You can use SELECT ... INTO with user-defined variables. The same syntax can also be used inside stored routines using cursors and local variables. See Section 13.2.10.1, “SELECT ... INTO Statement”.

#### 1.7.2.2 UPDATE Differences

​	If you access a column from the table to be updated in an expression, UPDATE uses the current value of the column. The second assignment in the following statement sets col2 to the current (updated) col1 value, not the original col1 value. The result is that col1 and col2 have the same value. This behavior differs from standard SQL.

```mysql
UPDATE t1 SET col1 = col1 + 1, col2 = col1;
```

#### 1.7.2.3 FOREIGN KEY Constraint Differences

​	The MySQL implementation of foreign key constraints differs from the SQL standard in the following key respects:

* If there are several rows in the parent table with the same referenced key value, InnoDB performs
  a foreign key check as if the other parent rows with the same key value do not exist. For example, if you define a RESTRICT type constraint, and there is a child row with several parent rows, InnoDB does not permit the deletion of any of the parent rows.
* If ON UPDATE CASCADE or ON UPDATE SET NULL recurses to update the same table it has previously updated during the same cascade, it acts like RESTRICT. This means that you cannot use self-referential ON UPDATE CASCADE or ON UPDATE SET NULL operations. This is to prevent infinite loops resulting from cascaded updates. A self-referential ON DELETE SET NULL, on the other hand, is possible, as is a self-referential ON DELETE CASCADE. Cascading operations may not be nested more than 15 levels deep.
* In an SQL statement that inserts, deletes, or updates many rows, foreign key constraints (like unique constraints) are checked row-by-row. When performing foreign key checks, InnoDB sets shared row level locks on child or parent records that it must examine. MySQL checks foreign key constraints immediately; the check is not deferred to transaction commit. According to the SQL standard, the default behavior should be deferred checking. That is, constraints are only checked after the entire SQL statement has been processed. This means that it is not possible to delete a row that refers to itself using a foreign key.
* No storage engine, including InnoDB, recognizes or enforces the MATCH clause used in referentialintegrity constraint definitions. Use of an explicit MATCH clause does not have the specified effect, and it causes ON DELETE and ON UPDATE clauses to be ignored. Specifying the MATCH should be avoided.
* MySQL requires that the referenced columns be indexed for performance reasons. However, MySQL does not enforce a requirement that the referenced columns be UNIQUE or be declared NOT NULL.
  * A FOREIGN KEY constraint that references a non-UNIQUE key is not standard SQL but rather an
    InnoDB extension. The NDB storage engine, on the other hand, requires an explicit unique key (or primary key) on any column referenced as a foreign key.
  * The handling of foreign key references to nonunique keys or keys that contain NULL values is not well defined for operations such as UPDATE or DELETE CASCADE. You are advised to use foreign keys that reference only UNIQUE (including PRIMARY) and NOT NULL keys.
* MySQL parses but ignores “inline REFERENCES specifications” (as defined in the SQL standard) where the references are defined as part of the column specification. MySQL accepts REFERENCES clauses only when specified as part of a separate FOREIGN KEY specification. For storage engines that do not support foreign keys (such as MyISAM), MySQL Server parses and ignores foreign key specifications.

#### 1.7.2.4 '--' as the Start of a Comment

​	Standard SQL uses the C syntax `/* this is a comment */` for comments, and MySQL Server supports this syntax as well. MySQL also support extensions to this syntax that enable MySQL-specific SQL to be embedded in the comment, as described in Section 9.7, “Comments”.

​	Standard SQL uses “--” as a start-comment sequence. MySQL Server uses # as the start comment character. MySQL Server also supports a variant of the -- comment style. That is, the -- startcomment sequence must be followed by a space (or by a control character such as a newline). The space is required to prevent problems with automatically generated SQL queries that use constructs such as the following, where we automatically insert the value of the payment for payment:

```java
UPDATE account SET credit=credit-payment
```

Consider about what happens if payment has a negative value such as -1:

```mysql
UPDATE account SET credit=credit--1
```

​	credit--1 is a valid expression in SQL, but -- is interpreted as the start of a comment, part of the expression is discarded. The result is a statement that has a completely different meaning than intended:

```mysql
UPDATE account SET credit=credit
```

### 1.7.3 How MySQL Deals with Constraints

#### 1.7.3.1 PRIMARY KEY and UNIQUE Index Constraints

​	Normally, errors occur for data-change statements (such as INSERT or UPDATE) that would violate primary-key, unique-key, or foreign-key constraints. If you are using a transactional storage engine such as InnoDB, MySQL automatically rolls back the statement. If you are using a nontransactional storage engine, MySQL stops processing the statement at the row for which the error occurred and leaves any remaining rows unprocessed.

​	MySQL supports an IGNORE keyword for INSERT, UPDATE, and so forth. If you use it, MySQL ignores primary-key or unique-key violations and continues processing with the next row. See the section for the statement that you are using (Section 13.2.6, “INSERT Statement”, Section 13.2.13, “UPDATE Statement”, and so forth).

​	You can get information about the number of rows actually inserted or updated with the mysql_info() C API function. You can also use the SHOW WARNINGS statement. See mysql_info(), and Section 13.7.7.42, “SHOW WARNINGS Statement”.

#### 1.7.3.2 FOREIGN KEY Constraints

​	MySQL supports ON UPDATE and ON DELETE foreign key references in CREATE TABLE and ALTER TABLE statements. The available referential actions are RESTRICT, CASCADE, SET NULL, and NO ACTION (the default).

​	SET DEFAULT is also supported by the MySQL Server but is currently rejected as invalid by InnoDB. Since MySQL does not support deferred constraint checking, NO ACTION is treated as RESTRICT. For the exact syntax supported by MySQL for foreign keys, see Section 13.1.20.5, “FOREIGN KEY Constraints”.

​	MySQL requires that foreign key columns be indexed; if you create a table with a foreign key constraint but no index on a given column, an index is created.

​	You can obtain information about foreign keys from the INFORMATION_SCHEMA.KEY_COLUMN_USAGE table. An example of a query against this table is shown here:

```mysql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME
> FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
> WHERE REFERENCED_TABLE_SCHEMA IS NOT NULL;
+--------------+---------------+-------------+-----------------+
| TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | CONSTRAINT_NAME |
+--------------+---------------+-------------+-----------------+
| fk1 | myuser | myuser_id | f |
| fk1 | product_order | customer_id | f2 |
| fk1 | product_order | product_id | f1 |
+--------------+---------------+-------------+-----------------+
3 rows in set (0.01 sec)
```

#### 1.7.3.3 Enforced Constraints on Invalid Data

​	By default, MySQL 8.0 rejects invalid or improper data values and aborts the statement in which they occur. It is possible to alter this behavior to be more forgiving of invalid values, such that the server coerces them to valid ones for data entry, by disabling strict SQL mode (see Section 5.1.11, “Server SQL Modes”), but this is not recommended.

#### 1.7.3.4 ENUM and SET Constraints

​	ENUM and SET columns provide an efficient way to define columns that can contain only a given set of values. See Section 11.3.5, “The ENUM Type”, and Section 11.3.6, “The SET Type”.

​	Unless strict mode is disabled (not recommended, but see Section 5.1.11, “Server SQL Modes”), the definition of a ENUM or SET column acts as a constraint on values entered into the column. An error occurs for values that do not satisfy these conditions:

* An ENUM value must be one of those listed in the column definition, or the internal numeric equivalent thereof. The value cannot be the error value (that is, 0 or the empty string). For a column defined as ENUM('a','b','c'), values such as '', 'd', or 'ax' are invalid and are rejected.
* A SET value must be the empty string or a value consisting only of the values listed in the column
  definition separated by commas. For a column defined as SET('a','b','c'), values such as 'd' or 'a,b,c,d' are invalid and are rejected.

## 1.8 Credits

略

# 3. Tutorial

To see a list of options provided by mysql, invoke it with the --help option:

```mysql
shell> mysql --help
```

## 3.1 Connecting to and Disconnecting from the Server

```mysql
shell> mysql -h host -u user -p
Enter password: ********
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 25338 to server version: 8.0.26-standard
Type 'help;' or '\h' for help. Type '\c' to clear the buffer.
mysql>
```

​	If you are logging in on the same machine that MySQL is running on, you can omit the host, and simply use the following:

```sh
shell> mysql -u user -p
```

​	If, when you attempt to log in, you get an error message such as ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2), it means that the MySQL server daemon (Unix) or service (Windows) is not running

​	For help with other problems often encountered when trying to log in, see Section B.3.2, “Common Errors When Using MySQL Programs”.

​	After you have connected successfully, you can disconnect any time by typing QUIT (or \q) at the

mysql> prompt:

```mysql
mysql> QUIT
Bye
```

​	On Unix, you can also disconnect by pressing Control+D.

## 3.2 Entering Queries

​	Here is a simple query that asks the server to tell you its version number and the current date. Type it in as shown here following the mysql> prompt and press Enter:

```mysql
mysql> SELECT VERSION(), CURRENT_DATE;
+-----------+--------------+
| VERSION() | CURRENT_DATE |
+-----------+--------------+
| 5.8.0-m17 | 2015-12-21 |
+-----------+--------------+
1 row in set (0.02 sec)
mysql>
```

This query illustrates several things about mysql:

* A query normally consists of an SQL statement followed by a semicolon. (There are some exceptions where a semicolon may be omitted. QUIT, mentioned earlier, is one of them. We'll get to others later.)
* When you issue a query, mysql sends it to the server for execution and displays the results, then prints another mysql> prompt to indicate that it is ready for another query.
* mysql displays query output in tabular form (rows and columns). The first row contains labels for the columns. The rows following are the query results. Normally, column labels are the names of the columns you fetch from database tables. If you're retrieving the value of an expression rather than a table column (as in the example just shown), mysql labels the column using the expression itself.
* mysql shows how many rows were returned and how long the query took to execute, which gives you a rough idea of server performance. These values are imprecise because they represent wall clock time (not CPU or machine time), and because they are affected by factors such as server load and network latency. (For brevity, the “rows in set” line is sometimes not shown in the remaining examples in this chapter.)

Keywords may be entered in any lettercase. The following queries are equivalent:

```mysql
mysql> SELECT VERSION(), CURRENT_DATE;
mysql> select version(), current_date;
mysql> SeLeCt vErSiOn(), current_DATE;
```

​	The following table shows each of the prompts you may see and summarizes what they mean about the state that mysql is in.

| Prompt | Meaning                                                      |
| ------ | ------------------------------------------------------------ |
| mysql> | Ready for new query                                          |
| ->     | Waiting for next line of multiple-line query                 |
| '>     | Waiting for next line, waiting for completion of a<br/>string that began with a single quote (') |
| ">     | Waiting for next line, waiting for completion of a<br/>string that began with a double quote (") |
| `>     | Waiting for next line, waiting for completion of an<br/>identifier that began with a backtick (`) |
| /*>    | Waiting for next line, waiting for completion of a<br/>comment that began with /* |

## 3.3 Creating and Using a Database

* Create a database
* Create a table
* Load data into the table
* Retrieve data from the table in various ways
* Use multiple tables

Use the SHOW statement to find out what databases currently exist on the server:

```mysql
mysql> SHOW DATABASES;
+----------+
| Database |
+----------+
| mysql |
| test |
| tmp |
+----------+
```

​	The `mysql` database describes user access privileges. The test database often is available as a workspace for users to try things out.

​	If the test database exists, try to access it:

```mysql
mysql> USE test
Database changed
```

The administrator needs to execute a statement like this:

```mysql
mysql> GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
```

### 3.3.1 Creating and Selecting a Database

```mysql
mysql> CREATE DATABASE menagerie;
```

​	Under Unix, database names are case-sensitive (unlike SQL keywords), so you must always refer to your database as menagerie, not as Menagerie, MENAGERIE, or some other variant. This is also true for table names. (Under Windows, this restriction does not apply, although you must refer to databases and tables using the same lettercase throughout a given query. However, for a variety of reasons, the recommended best practice is always to use the same lettercase that was used when the database was created.)

> If you get an error such as ERROR 1044 (42000): Access denied for user 'micah'@'localhost' to database 'menagerie' when attempting to create a database, this means that your user account does not have the necessary privileges to do so. Discuss this with the administrator or see Section 6.2, “Access Control and Account Management”.

```mysql
shell> mysql -h host -u user -p menagerie
Enter password: ********
```

### 3.3.2 Creating a Table

```mysql
mysql> SHOW TABLES;
Empty set (0.00 sec)
```

Use a CREATE TABLE statement to specify the layout of your table:

```mysql
mysql> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
```

​	If you make a poor choice and it turns out later that you need a longer field, MySQL provides an ALTER TABLE statement.

​	To verify that your table was created the way you expected, use a DESCRIBE statement:

```mysql
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name | varchar(20) | YES | | NULL | |
| owner | varchar(20) | YES | | NULL | |
| species | varchar(20) | YES | | NULL | |
| sex | char(1) | YES | | NULL | |
| birth | date | YES | | NULL | |
| death | date | YES | | NULL | |
+---------+-------------+------+-----+---------+-------+
```

### 3.3.3 Loading Data into a Table

​	You could create a text file pet.txt containing one record per line, with values separated by tabs, and given in the order in which the columns were listed in the CREATE TABLE statement. For missing values (such as unknown sexes or death dates for animals that are still living), you can use NULL values. To represent these in your text file, use \N (backslash, capital-N). For example, the record for Whistler the bird would look like this (where the whitespace between values is a single tab character):

```mysql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
```

​	If you created the file on Windows with an editor that uses \r\n as a line terminator, you should use this statement instead:

```mysql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
LINES TERMINATED BY '\r\n';
```

(On an Apple machine running macOS, you would likely want to use LINES TERMINATED BY '\r'.)

### 3.3.4 Retrieving Information from a Table

```mysql
SELECT what_to_select
FROM which_table
WHERE conditions_to_satisfy;
```

#### 3.3.4.1 Selecting All Data

```mysql
mysql> SELECT * FROM pet;
+----------+--------+---------+------+------------+------------+
| name | owner | species | sex | birth | death |
+----------+--------+---------+------+------------+------------+
| Fluffy | Harold | cat | f | 1993-02-04 | NULL |
| Claws | Gwen | cat | m | 1994-03-17 | NULL |
| Buffy | Harold | dog | f | 1989-05-13 | NULL |
| Fang | Benny | dog | m | 1990-08-27 | NULL |
| Bowser | Diane | dog | m | 1979-08-31 | 1995-07-29 |
| Chirpy | Gwen | bird | f | 1998-09-11 | NULL |
| Whistler | Gwen | bird | NULL | 1997-12-09 | NULL |
| Slim | Benny | snake | m | 1996-04-29 | NULL |
| Puffball | Diane | hamster | f | 1999-03-30 | NULL |
+----------+--------+---------+------+------------+------------+
```

#### 3.3.4.2 Selecting Particular Rows

略

#### 3.3.4.3 Selecting Particular Columns

略

#### 3.3.4.4 Sorting Rows

略

#### 3.3.4.5 Date Calculations

* TIMESTAMPDIFF(YEAR,birth,CURDATE())
* YEAR(), MONTH(), and DAYOFMONTH(). MONTH()
* DATE_ADD(CURDATE(),INTERVAL 1 MONTH)
* MOD(something,12)
* SELECT '2018-10-31' + INTERVAL 1 DAY;

```mysql
mysql> SELECT '2018-10-31' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-31' + INTERVAL 1 DAY |
+-------------------------------+
| 2018-11-01 |
+-------------------------------+
mysql> SELECT '2018-10-32' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-32' + INTERVAL 1 DAY |
+-------------------------------+
| NULL |
+-------------------------------+
mysql> SHOW WARNINGS;
+---------+------+----------------------------------------+
| Level | Code | Message |
+---------+------+----------------------------------------+
| Warning | 1292 | Incorrect datetime value: '2018-10-32' |
+---------+------+----------------------------------------+
```

#### 3.3.4.6 Working with NULL Values

```mysql
mysql> SELECT 1 IS NULL, 1 IS NOT NULL;
+-----------+---------------+
| 1 IS NULL | 1 IS NOT NULL |
+-----------+---------------+
| 0 | 1 |

mysql> SELECT 1 = NULL, 1 <> NULL, 1 < NULL, 1 > NULL;
+----------+-----------+----------+----------+
| 1 = NULL | 1 <> NULL | 1 < NULL | 1 > NULL |
+----------+-----------+----------+----------+
| NULL | NULL | NULL | NULL |
+----------+-----------+----------+----------+
```

​	In MySQL, 0 or NULL means false and anything else means true. The default truth value from a boolean operation is 1.

​	Two NULL values are regarded as equal in a GROUP BY.

```mysql
mysql> SELECT 0 IS NULL, 0 IS NOT NULL, '' IS NULL, '' IS NOT NULL;
+-----------+---------------+------------+----------------+
| 0 IS NULL | 0 IS NOT NULL | '' IS NULL | '' IS NOT NULL |
+-----------+---------------+------------+----------------+
| 0 | 1 | 0 | 1 |
+-----------+---------------+------------+----------------+
```

#### 3.3.4.7 Pattern Matching

​	MySQL provides standard SQL pattern matching as well as a form of pattern matching based on extended regular expressions similar to those used by Unix utilities such as vi, grep, and sed.

```mysql
SELECT * FROM pet WHERE name LIKE 'b%';
```

​	To find names containing exactly five characters, use five instances of the _ pattern character:

```mysql
mysql> SELECT * FROM pet WHERE name LIKE '_____';
+-------+--------+---------+------+------------+-------+
| name | owner | species | sex | birth | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen | cat | m | 1994-03-17 | NULL |
| Buffy | Harold | dog | f | 1989-05-13 | NULL |
+-------+--------+---------+------+------------+-------+
```

* . matches any single character.
* A character class [...] matches any character within the brackets. For example, [abc] matches
  a, b, or c. To name a range of characters, use a dash. [a-z] matches any letter, whereas [0-9]
  matches any digit.
* matches zero or more instances of the thing preceding it. For example, x* matches any number of
  x characters, [0-9]* matches any number of digits, and .* matches any number of anything.
* A regular expression pattern match succeeds if the pattern matches anywhere in the value being tested. (This differs from a LIKE pattern match, which succeeds only if the pattern matches the entire value.)
* To anchor a pattern so that it must match the beginning or end of the value being tested, use ^ at the beginning or $ at the end of the pattern.



​	To demonstrate how extended regular expressions work, the LIKE queries shown previously are rewritten here to use REGEXP_LIKE().

```mysql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b');
+--------+--------+---------+------+------------+------------+
| name | owner | species | sex | birth | death |
+--------+--------+---------+------+------------+------------+
| Buffy | Harold | dog | f | 1989-05-13 | NULL |
| Bowser | Diane | dog | m | 1979-08-31 | 1995-07-29 |
+--------+--------+---------+------+------------+------------+
```

​	To force a regular expression comparison to be case-sensitive, use a case-sensitive collation, or use the BINARY keyword to make one of the strings a binary string, or specify the c match-control character. Each of these queries matches only lowercase b at the beginning of a name:

```mysql
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b' COLLATE utf8mb4_0900_as_cs);
SELECT * FROM pet WHERE REGEXP_LIKE(name, BINARY '^b');
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b', 'c');
```

#### 3.3.4.8 Counting Rows

```mysql
SELECT COUNT(*) FROM pet;
```

Earlier, you retrieved the names of the people who owned pets. You can use COUNT() if you want to
find out how many pets each owner has:

```mysql
SELECT owner, COUNT(*) FROM pet GROUP BY owner;
```

​	If you name columns to select in addition to the COUNT() value, a GROUP BY clause should be present that names those same columns. Otherwise, the following occurs:

* If the ONLY_FULL_GROUP_BY SQL mode is enabled, an error occurs:

```mysql
mysql> SET sql_mode = 'ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT owner, COUNT(*) FROM pet;
ERROR 1140 (42000): In aggregated query without GROUP BY, expression
#1 of SELECT list contains nonaggregated column 'menagerie.pet.owner';
this is incompatible with sql_mode=only_full_group_by
```

* If ONLY_FULL_GROUP_BY is not enabled, the query is processed by treating all rows as a single group, but the value selected for each named column is nondeterministic. The server is free to select the value from any row:

```mysql
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT owner, COUNT(*) FROM pet;
+--------+----------+
| owner | COUNT(*) |
+--------+----------+
| Harold | 8 |
+--------+----------+
1 row in set (0.00 sec)
```

#### 3.3.4.9 Using More Than one Table

```mysql
mysql> CREATE TABLE event (name VARCHAR(20), date DATE,
type VARCHAR(15), remark VARCHAR(255));
```

![](https://pic.imgdb.cn/item/6120b57f4907e2d39c8ccd94.jpg)

```mysql
mysql> SELECT pet.name,
TIMESTAMPDIFF(YEAR,birth,date) AS age,
remark
FROM pet INNER JOIN event
ON pet.name = event.name
WHERE event.type = 'litter';
+--------+------+-----------------------------+
| name | age | remark |
+--------+------+-----------------------------+
| Fluffy | 2 | 4 kittens, 3 female, 1 male |
| Buffy | 4 | 5 puppies, 2 female, 3 male |
| Buffy | 5 | 3 puppies, 3 female |
+--------+------+-----------------------------+
```

## 3.4 Getting Information About Databases and Tables

You have previously seen SHOW DATABASES, which lists the databases managed by the server. To find out which database is currently selected, use the DATABASE() function:

```mysql
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| menagerie |
+------------+
```

```mysql
mysql> SHOW TABLES;
+---------------------+
| Tables_in_menagerie |
+---------------------+
| event |
| pet |
+---------------------+
```

The name of the column in the output produced by this statement is always Tables_in_db_name, where db_name is the name of the database. See Section 13.7.7.39, “SHOW TABLES Statement”, for more information.

​	If you want to find out about the structure of a table, the DESCRIBE statement is useful; it displays information about each of a table's columns:

```mysql
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name | varchar(20) | YES | | NULL | |
| owner | varchar(20) | YES | | NULL | |
| species | varchar(20) | YES | | NULL | |
| sex | char(1) | YES | | NULL | |
| birth | date | YES | | NULL | |
| death | date | YES | | NULL | |
+---------+-------------+------+-----+---------+-------+
```

## 3.5 Using mysql in Batch Mode

​	You can also run mysql in batch mode. To do this, put the statements you want to run in a file, then tell mysql to read its input from the file:

```shell
shell> mysql < batch-file
```

​	If you are running mysql under Windows and have some special characters in the file that cause problems, you can do this:

```shell
C:\> mysql -e "source batch-file"
```

```shell
shell> mysql -h host -u user -p < batch-file
Enter password: ********
```

​	When you use mysql this way, you are creating a script file, then executing the script.

​	If you want the script to continue even if some of the statements in it produce errors, you should use the `--force` command-line option.

* If you run a query repeatedly (say, every day or every week), making it a script enables you to avoid retyping it each time you execute it.
* You can generate new queries from existing ones that are similar by copying and editing script files.
* Batch mode can also be useful while you're developing a query, particularly for multiple-line statements or multiple-statement sequences. If you make a mistake, you don't have to retype everything. Just edit your script to correct the error, then tell mysql to execute it again.
* If you have a query that produces a lot of output, you can run the output through a pager rather than watching it scroll off the top of your screen:

```shell
shell> mysql < batch-file | more
```

* You can catch the output in a file for further processing:

```mysql
shell> mysql < batch-file > mysql.out
```

* You can distribute your script to other people so that they can also run the statements.
* Some situations do not allow for interactive use, for example, when you run a query from a cron job.
  In this case, you must use batch mode.



​	If you want to get the interactive output format in batch mode, use `mysql -t`. To echo to the output the statements that are executed, use `mysql -v`.

​	You can also use scripts from the mysql prompt by using the source command or \. command:

```mysql
mysql> source filename;
mysql> \. filename
```

## 3.6 Examples of Common Queries

![](https://pic.imgdb.cn/item/6120c6fe4907e2d39cb1f266.jpg)

### 3.6.1 The Maximum Value for a Column

```mysql
SELECT MAX(article) AS article FROM shop;
+---------+
| article |
+---------+
| 4 |
+---------+
```

### 3.6.2 The Row Holding the Maximum of a Certain Column

```mysql
SELECT article, dealer, price
FROM shop
WHERE price=(SELECT MAX(price) FROM shop);
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
| 0004 | D | 19.95 |
+---------+--------+-------+
```

​	Other solutions are to use a LEFT JOIN or to sort all rows descending by price and get only the first row using the MySQL-specific LIMIT clause:

```mysql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.price < s2.price
WHERE s2.article IS NULL;
SELECT article, dealer, price
FROM shop
ORDER BY price DESC
LIMIT 1;
```

### 3.6.3 Maximum of Column per Group

```mysql
SELECT article, MAX(price) AS price
FROM shop
GROUP BY article
ORDER BY article;

+---------+-------+
| article | price |
+---------+-------+
| 0001 | 3.99 |
| 0002 | 10.99 |
| 0003 | 1.69 |
| 0004 | 19.95 |
+---------+-------+
```

### 3.6.4 The Rows Holding the Group-wise Maximum of a Certain Column

```mysql
SELECT article, dealer, price
FROM shop s1
WHERE price=(SELECT MAX(s2.price)
FROM shop s2
WHERE s1.article = s2.article)
ORDER BY article;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
| 0001 | B | 3.99 |
| 0002 | A | 10.99 |
| 0003 | C | 1.69 |
| 0004 | D | 19.95 |
+---------+--------+-------+
```

Common table expression with window function:

```mysql
WITH s1 AS (
SELECT article, dealer, price,
RANK() OVER (PARTITION BY article
ORDER BY price DESC
) AS `Rank`
FROM shop
)
SELECT article, dealer, price
FROM s1
WHERE `Rank` = 1
ORDER BY article;
```

**LEFT JOIN:**

```mysql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.article = s2.article AND s1.price < s2.price
WHERE s2.article IS NULL
ORDER BY s1.article;
```

### 3.6.5 Using User-Defined Variables

```mysql
mysql> SELECT @min_price:=MIN(price),@max_price:=MAX(price) FROM shop;
mysql> SELECT * FROM shop WHERE price=@min_price OR price=@max_price;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
| 0003 | D | 1.25 |
| 0004 | D | 19.95 |
+---------+--------+-------+
```

> It is also possible to store the name of a database object such as a table or a column in a user variable and then to use this variable in an SQL statement; however, this requires the use of a prepared statement. See Section 13.5, “Prepared Statements”, for more information.

### 3.6.6 Using Foreign Keys

```mysql
CREATE TABLE person (
id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
name CHAR(60) NOT NULL,
PRIMARY KEY (id)
);
CREATE TABLE shirt (
id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
style ENUM('t-shirt', 'polo', 'dress') NOT NULL,
color ENUM('red', 'blue', 'orange', 'white', 'black') NOT NULL,
owner SMALLINT UNSIGNED NOT NULL REFERENCES person(id),
PRIMARY KEY (id)
);
INSERT INTO person VALUES (NULL, 'Antonio Paz');
SELECT @last := LAST_INSERT_ID();
INSERT INTO shirt VALUES
(NULL, 'polo', 'blue', @last),
(NULL, 'dress', 'white', @last),
(NULL, 't-shirt', 'blue', @last);
INSERT INTO person VALUES (NULL, 'Lilliana Angelovska');
SELECT @last := LAST_INSERT_ID();
INSERT INTO shirt VALUES
(NULL, 'dress', 'orange', @last),
(NULL, 'polo', 'red', @last),
(NULL, 'dress', 'blue', @last),
(NULL, 't-shirt', 'white', @last);
SELECT * FROM person;
+----+---------------------+
| id | name |
+----+---------------------+
| 1 | Antonio Paz |
| 2 | Lilliana Angelovska |
+----+---------------------+
SELECT * FROM shirt;
+----+---------+--------+-------+
| id | style | color | owner |
+----+---------+--------+-------+
| 1 | polo | blue | 1 |
| 2 | dress | white | 1 |
| 3 | t-shirt | blue | 1 |
| 4 | dress | orange | 2 |
| 5 | polo | red | 2 |
| 6 | dress | blue | 2 |
| 7 | t-shirt | white | 2 |
+----+---------+--------+-------+
SELECT s.* FROM person p INNER JOIN shirt s
ON s.owner = p.id
WHERE p.name LIKE 'Lilliana%'
AND s.color <> 'white';
+----+-------+--------+-------+
| id | style | color | owner |
+----+-------+--------+-------+
| 4 | dress | orange | 2 |
| 5 | polo | red | 2 |
| 6 | dress | blue | 2 |
+----+-------+--------+-------+
```

```mysql
SHOW CREATE TABLE shirt\G
*************************** 1. row ***************************
Table: shirt
Create Table: CREATE TABLE `shirt` (
`id` smallint(5) unsigned NOT NULL auto_increment,
`style` enum('t-shirt','polo','dress') NOT NULL,
`color` enum('red','blue','orange','white','black') NOT NULL,
`owner` smallint(5) unsigned NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4
```

### 3.6.7 Searching on Two Keys

```mysql
SELECT field1_index, field2_index FROM test_table
WHERE field1_index = '1' OR field2_index = '1'
```

### 3.6.8 Calculating Visits Per Day

```mysql
CREATE TABLE t1 (year YEAR, month INT UNSIGNED,
day INT UNSIGNED);
INSERT INTO t1 VALUES(2000,1,1),(2000,1,20),(2000,1,30),(2000,2,2),
(2000,2,23),(2000,2,23);

SELECT year,month,BIT_COUNT(BIT_OR(1<<day)) AS days FROM t1
GROUP BY year,month;
```

### 3.6.9 Using AUTO_INCREMENT

```mysql
CREATE TABLE animals (
id MEDIUMINT NOT NULL AUTO_INCREMENT,
name CHAR(30) NOT NULL,
PRIMARY KEY (id)
);
INSERT INTO animals (name) VALUES
('dog'),('cat'),('penguin'),
('lax'),('whale'),('ostrich');
SELECT * FROM animals;
```

​	No value was specified for the AUTO_INCREMENT column, so MySQL assigned sequence numbers automatically. You can also explicitly assign 0 to the column to generate sequence numbers, unless the NO_AUTO_VALUE_ON_ZERO SQL mode is enabled. For example:

```mysql
INSERT INTO animals (id,name) VALUES(0,'groundhog');
```



​	When you insert any other value into an AUTO_INCREMENT column, the column is set to that value and the sequence is reset so that the next automatically generated value follows sequentially from the largest column value. For example:

```mysql
INSERT INTO animals (id,name) VALUES(100,'rabbit');
INSERT INTO animals (id,name) VALUES(NULL,'mouse');
SELECT * FROM animals;
+-----+-----------+
| id | name |
+-----+-----------+
| 1 | dog |
| 2 | cat |
| 3 | penguin |
| 4 | lax |
| 5 | whale |
| 6 | ostrich |
| 7 | groundhog |
| 8 | squirrel |
| 100 | rabbit |
| 101 | mouse |
+-----+-----------+
```

​	You can retrieve the most recent automatically generated AUTO_INCREMENT value with the LAST_INSERT_ID() SQL function or the mysql_insert_id() C API function. These functions are connection-specific, so their return values are not affected by another connection which is also performing inserts.

> For a multiple-row insert, LAST_INSERT_ID() and mysql_insert_id() actually return the AUTO_INCREMENT key from the first of the inserted rows. This enables multiple-row inserts to be reproduced correctly on other servers in a replication setup.



​	To start with an AUTO_INCREMENT value other than 1, set that value with CREATE TABLE or ALTER TABLE, like this:

```mysql
mysql> ALTER TABLE tbl AUTO_INCREMENT = 100;
```

**InnoDB Notes**

​	For information about AUTO_INCREMENT usage specific to InnoDB, see Section 15.6.1.6, “AUTO_INCREMENT Handling in InnoDB”.

**MyISAM Notes**

* For MyISAM tables, you can specify AUTO_INCREMENT on a secondary column in a multiplecolumn index. In this case, the generated value for the AUTO_INCREMENT column is calculated as MAX(auto_increment_column) + 1 WHERE prefix=given-prefix. This is useful when you want to put data into ordered groups.

```mysql
CREATE TABLE animals (
grp ENUM('fish','mammal','bird') NOT NULL,
id MEDIUMINT NOT NULL AUTO_INCREMENT,
name CHAR(30) NOT NULL,
PRIMARY KEY (grp,id)
) ENGINE=MyISAM;
INSERT INTO animals (grp,name) VALUES
('mammal','dog'),('mammal','cat'),
('bird','penguin'),('fish','lax'),('mammal','whale'),
('bird','ostrich');
SELECT * FROM animals ORDER BY grp,id;
+--------+----+---------+
| grp | id | name |
+--------+----+---------+
| fish | 1 | lax |
| mammal | 1 | dog |
| mammal | 2 | cat |
| mammal | 3 | whale |
| bird | 1 | penguin |
| bird | 2 | ostrich |
+--------+----+---------+
```

* If the AUTO_INCREMENT column is part of multiple indexes, MySQL generates sequence values using the index that begins with the AUTO_INCREMENT column, if there is one. For example, if the animals table contained indexes PRIMARY KEY (grp, id) and INDEX (id), MySQL would ignore the PRIMARY KEY for generating sequence values. As a result, the table would contain a single sequence, not a sequence per grp value.

**Further Reading**

More information about AUTO_INCREMENT is available here:

* How to assign the AUTO_INCREMENT attribute to a column: Section 13.1.20, “CREATE TABLE Statement”, and Section 13.1.9, “ALTER TABLE Statement”.
* How AUTO_INCREMENT behaves depending on the NO_AUTO_VALUE_ON_ZERO SQL mode: Section 5.1.11, “Server SQL Modes”.
* How to use the LAST_INSERT_ID() function to find the row that contains the most recent AUTO_INCREMENT value: Section 12.16, “Information Functions”.
* Setting the AUTO_INCREMENT value to be used: Section 5.1.8, “Server System Variables”.
* Section 15.6.1.6, “AUTO_INCREMENT Handling in InnoDB”
* AUTO_INCREMENT and replication: Section 17.5.1.1, “Replication and AUTO_INCREMENT”.
* Server-system variables related to AUTO_INCREMENT (auto_increment_increment and auto_increment_offset) that can be used for replication: Section 5.1.8, “Server System Variables”.

## 3.7 Using MySQL with Apache

You can change the Apache logging format to be easily readable by MySQL by putting the following into the Apache configuration file:

```mysql
LogFormat \
"\"%h\",%{%Y%m%d%H%M%S}t,%>s,\"%b\",\"%{Content-Type}o\", \
\"%U\",\"%{Referer}i\",\"%{User-Agent}i\""

```

```mysql
LOAD DATA INFILE '/local/access_log' INTO TABLE tbl_name
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\'
```

# 5. MySQL Server Administration

* Server configuration
* The data directory, particularly the mysql system schema
* The server log files
* Management of multiple servers on a single machine

## 5.1 The MySQL Server

​	mysqld is the MySQL server. The following discussion covers these MySQL server configuration topics:

* Startup options that the server supports. You can specify these options on the command line, through configuration files, or both.
* Server system variables. These variables reflect the current state and values of the startup options, some of which can be modified while the server is running.
* Server status variables. These variables contain counters and statistics about runtime operation.
* How to set the server SQL mode. This setting modifies certain aspects of SQL syntax and semantics, for example for compatibility with code from other database systems, or to control the error handling for particular situations.
* How the server manages client connections.
* Configuring and using IPv6 and network namespace support.
* Configuring and using time zone support.
* Using resource groups.
* Server-side help capabilities.
* Capabilities provided to enable client session state changes.
* The server shutdown process. There are performance and reliability considerations depending on
  the type of table (transactional or nontransactional) and whether you use replication.

> Not all storage engines are supported by all MySQL server binaries and configurations. To find out how to determine which storage engines your MySQL server installation supports, see Section 13.7.7.16, “SHOW ENGINES Statement”.

### 5.1.1 Configuring the Server

The MySQL server, mysqld, has many command options and system variables that can be set at startup to configure its operation. To determine the default command option and system variable values used by the server, execute this command:

```mysql
shell> mysqld --verbose --help

abort-slave-event-count 0
allow-suspicious-udfs FALSE
archive ON
auto-increment-increment 1
auto-increment-offset 1
autocommit TRUE
automatic-sp-privileges TRUE
avoid-temporal-upgrade FALSE
back-log 80
basedir /home/jon/bin/mysql-8.0/
...
tmpdir /tmp
transaction-alloc-block-size 8192
transaction-isolation REPEATABLE-READ
transaction-prealloc-size 4096
transaction-read-only FALSE
transaction-write-set-extraction XXHASH64
updatable-views-with-limit YES
validate-user-plugins TRUE
verbose TRUE
wait-timeout 28800
```

​	To see the current system variable values actually used by the server as it runs, connect to it and execute this statement:

```mysql
mysql> SHOW VARIABLES;
```

To see some statistical and status indicators for a running server, execute this statement:

```mysql
mysql> SHOW STATUS;
```

System variable and status information also is available using the mysqladmin command:

```shell
shell> mysqladmin variables
shell> mysqladmin extended-status
```

### 5.1.2 Server Configuration Defaults

​	The MySQL server has many operating parameters, which you can change at server startup using command-line options or configuration files (option files). It is also possible to change many parameters at runtime. For general instructions on setting parameters at startup or runtime, see [Section 5.1.7, “Server Command Options”](https://dev.mysql.com/doc/refman/8.0/en/server-options.html), and [Section 5.1.8, “Server System Variables”](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html).

​	On Windows, MySQL Installer interacts with the user and creates a file named `my.ini` in the base installation directory as the default option file.

> On Windows, the `.ini` or `.cnf` option file extension might not be displayed.



​	After completing the installation process, you can edit the default option file at any time to modify the parameters used by the server. For example, to use a parameter setting in the file that is commented with a `#` character at the beginning of the line, remove the `#`, and modify the parameter value if necessary. To disable a setting, either add a `#` to the beginning of the line or remove it.

​	For non-Windows platforms, no default option file is created during either the server installation or the data directory initialization process. Create your option file by following the instructions given in [Section 4.2.2.2, “Using Option Files”](https://dev.mysql.com/doc/refman/8.0/en/option-files.html). Without an option file, the server just starts with its default settings—see [Section 5.1.2, “Server Configuration Defaults”](https://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html) on how to check those settings.

​	For additional information about option file format and syntax, see [Section 4.2.2.2, “Using Option Files”](https://dev.mysql.com/doc/refman/8.0/en/option-files.html).

### 5.1.3 Server Configuration Validation

​	As of MySQL 8.0.16, MySQL Server supports a [`--validate-config`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_validate-config) option that enables the startup configuration to be checked for problems without running the server in normal operational mode:

​	If no errors are found, the server terminates with an exit code of 0. If an error is found, the server displays a diagnostic message and terminates with an exit code of 1. For example:

```shell
shell> mysqld --validate-config --no-such-option
2018-11-05T17:50:12.738919Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T17:50:12.738962Z 0 [ERROR] [MY-010119] [Server] Aborting
```

​	For example, this command produces multiple warnings, both of which are displayed. But no error occurs, so the exit code is 0:

```shell
shell> mysqld --validate-config --log_error_verbosity=2
         --read-only=s --transaction_read_only=s
2018-11-05T15:43:18.445863Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:18.445882Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
```

This command produces the same warnings, but also an error, so the error message is displayed along with the warnings and the exit code is 1:

```shell
shell> mysqld --validate-config --log_error_verbosity=2
         --no-such-option --read-only=s --transaction_read_only=s
2018-11-05T15:43:53.152886Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.152913Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.164889Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T15:43:53.165053Z 0 [ERROR] [MY-010119] [Server] Aborting
```

[`--validate-config`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_validate-config) can be used with the [`--defaults-file`](https://dev.mysql.com/doc/refman/8.0/en/option-file-options.html#option_general_defaults-file) option to validate only the options in a specific file:

```mysql
shell> mysqld --defaults-file=./my.cnf-test --validate-config
2018-11-05T10:40:02.712141Z 0 [ERROR] [MY-000067] [Server] unknown variable
'tx_read_only=ON'.
2018-11-05T10:40:02.712178Z 0 [ERROR] [MY-010119] [Server] Aborting
```

### 5.1.4 Server Option, System Variable, and Status Variable Reference

​	The table lists command-line options (Cmd-line), options valid in configuration files (Option file), server system variables (System Var), and status variables (Status var) in one unified list, with an indication of where each option or variable is valid. If a server option set on the command line or in an option file differs from the name of the corresponding system variable, the variable name is noted immediately below the corresponding option. For system and status variables, the scope of the variable (Var Scope) is Global, Session, or both. Please see the corresponding item descriptions for details on setting and using the options and variables. Where appropriate, direct links to further information about the items are provided.

​	For a version of this table that is specific to NDB Cluster, see [Section 23.4.2.5, “NDB Cluster mysqld Option and Variable Reference”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-option-tables.html).

**Table 5.1 Command-Line Option, System Variable, and Status Variable Summary**

**Notes:**

https://dev.mysql.com/doc/refman/8.0/en/server-option-variable-reference.html

1. This option is dynamic, but should be set only by server. You should not set this variable manually.

### 5.1.5 Server System Variable Reference

​	The following table lists all system variables applicable within `mysqld`.

​	The table lists command-line options (Cmd-line), options valid in configuration files (Option file), server system variables (System Var), and status variables (Status var) in one unified list, with an indication of where each option or variable is valid. If a server option set on the command line or in an option file differs from the name of the corresponding system variable, the variable name is noted immediately below the corresponding option. The scope of the variable (Var Scope) is Global, Session, or both. Please see the corresponding item descriptions for details on setting and using the variables. Where appropriate, direct links to further information about the items are provided.

**Table 5.2 System Variable Summary**

https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html

**Notes:**

​	1. This option is dynamic, but should be set only by server. You should not set this variable manually.

### 5.1.6 Server Status Variable Reference

​	The following table lists all status variables applicable within `mysqld`.

​	The table lists each variable's data type and scope. The last column indicates whether the scope for each variable is Global, Session, or both. Please see the corresponding item descriptions for details on setting and using the variables. Where appropriate, direct links to further information about the items are provided.

**Table 5.3 Status Variable Summary**

https://dev.mysql.com/doc/refman/8.0/en/server-status-variable-reference.html

### 5.1.7 Server Command Options

https://dev.mysql.com/doc/refman/8.0/en/server-options.html

​	When you start the [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) server, you can specify program options using any of the methods described in [Section 4.2.2, “Specifying Program Options”](https://dev.mysql.com/doc/refman/8.0/en/program-options.html). The most common methods are to provide options in an option file or on the command line. However, in most cases it is desirable to make sure that the server uses the same options each time it runs. The best way to ensure this is to list them in an option file. See [Section 4.2.2.2, “Using Option Files”](https://dev.mysql.com/doc/refman/8.0/en/option-files.html). That section also describes option file format and syntax.

​	[**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) reads options from the `[mysqld]` and `[server]` groups. [**mysqld_safe**](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html) reads options from the `[mysqld]`, `[server]`, `[mysqld_safe]`, and `[safe_mysqld]` groups. [**mysql.server**](https://dev.mysql.com/doc/refman/8.0/en/mysql-server.html) reads options from the `[mysqld]` and `[mysql.server]` groups.

​	[**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) accepts many command options. For a brief summary, execute this command:

```mysql
mysqld --help
# To see the full list, use this command:
mysqld --verbose --help
```

The following list shows some of the most common server options. Additional options are described in other sections:

- Options that affect security: See [Section 6.1.4, “Security-Related mysqld Options and Variables”](https://dev.mysql.com/doc/refman/8.0/en/security-options.html).
- SSL-related options: See [Command Options for Encrypted Connections](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#encrypted-connection-options).
- Binary log control options: See [Section 5.4.4, “The Binary Log”](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html).
- Replication-related options: See [Section 17.1.6, “Replication and Binary Logging Options and Variables”](https://dev.mysql.com/doc/refman/8.0/en/replication-options.html).
- Options for loading plugins such as pluggable storage engines: See [Section 5.6.1, “Installing and Uninstalling Plugins”](https://dev.mysql.com/doc/refman/8.0/en/plugin-loading.html).
- Options specific to particular storage engines: See [Section 15.14, “InnoDB Startup Options and System Variables”](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html) and [Section 16.2.1, “MyISAM Startup Options”](https://dev.mysql.com/doc/refman/8.0/en/myisam-start.html).



​	Some options control the size of buffers or caches. For a given buffer, the server might need to allocate internal data structures. These structures typically are allocated from the total memory allocated to the buffer, and the amount of space required might be platform dependent. This means that when you assign a value to an option that controls a buffer size, the amount of space actually available might differ from the value assigned. In some cases, the amount might be less than the value assigned. It is also possible that the server adjusts a value upward. For example, if you assign a value of 0 to an option for which the minimal value is 1024, the server sets the value to 1024.

​	Values for buffer sizes, lengths, and stack sizes are given in bytes unless otherwise specified.

​	You can also set the values of server system variables at server startup by using variable names as options. To assign a value to a server system variable, use an option of the form `--var_name=value`. For example, [`--sort_buffer_size=384M`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sort_buffer_size) sets the [`sort_buffer_size`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sort_buffer_size) variable to a value of 384MB.

​	To restrict the maximum value to which a system variable can be set at runtime with the [`SET`](https://dev.mysql.com/doc/refman/8.0/en/set-variable.html) statement, specify this maximum by using an option of the form `--maximum-*`var_name`*=*`value`*` at server startup.

* [`--help`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_help), `-?`

| Command-Line Format | `--help` |
| :------------------ | -------- |

​	Display a short help message and exit. Use both the [`--verbose`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_verbose) and [`--help`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_help) options to see the full message.

* [`--admin-ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl), [`--skip-admin-ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl)

| Command-Line Format | `--admin-ssl[={OFF|ON}]` |
| :------------------ | ------------------------ |
| Introduced          | 8.0.21                   |
| Deprecated          | 8.0.26                   |
| Type                | Boolean                  |
| Default Value       | `ON`                     |

​	The [`--admin-ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl) option is like the [`--ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_ssl) option, except that it applies to the administrative connection interface rather than the main connection interface. For information about these interfaces, see [Section 5.1.12.1, “Connection Interfaces”](https://dev.mysql.com/doc/refman/8.0/en/connection-interfaces.html).

​	The [`--admin-ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl) option specifies that the server permits but does not require encrypted connections on the administrative interface. This option is enabled by default.

​	`--admin-ssl` can be specified in negated form as `--skip-admin-ssl` or a synonym ([`--admin-ssl=OFF`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl), [`--disable-admin-ssl`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl)). In this case, the option specifies that the server does *not* permit encrypted connections, regardless of the settings of the `admin_tsl_*`xxx`*` and `admin_ssl_*`xxx`*` system variables.

​	Set the [`admin_tls_version`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_tls_version) system variable to the empty value to indicate that no TLS versions are supported. For example, these lines in the server `my.cnf` file disable encrypted connections:

```
[mysqld]
admin_tls_version=''
```

* [`--allow-suspicious-udfs`]

| Command-Line Format | `--allow-suspicious-udfs[={OFF|ON}]` |
| :------------------ | ------------------------------------ |
| Type                | Boolean                              |
| Default Value       | `OFF`                                |

​	This option controls whether loadable functions that have only an `xxx` symbol for the main function can be loaded. By default, the option is off and only loadable functions that have at least one auxiliary symbol can be loaded; this prevents attempts at loading functions from shared object files other than those containing legitimate functions. See [Loadable Function Security Precautions](https://dev.mysql.com/doc/extending-mysql/8.0/en/adding-loadable-function.html#loadable-function-security).

* [`--ansi`]

| Command-Line Format | `--ansi` |
| :------------------ | -------- |

​	Use standard (ANSI) SQL syntax instead of MySQL syntax. For more precise control over the server SQL mode, use the [`--sql-mode`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sql-mode) option instead. See [Section 1.7, “MySQL Standards Compliance”](https://dev.mysql.com/doc/refman/8.0/en/compatibility.html), and [Section 5.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html).

* [`--basedir=*`dir_name`*`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_basedir), [`-b *`dir_name`*`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_basedir)

| Command-Line Format                                          | `--basedir=dir_name`                      |
| :----------------------------------------------------------- | ----------------------------------------- |
| System Variable                                              | `basedir`                                 |
| Scope                                                        | Global                                    |
| Dynamic                                                      | No                                        |
| [`SET_VAR`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-set-var) Hint Applies | No                                        |
| Type                                                         | Directory name                            |
| Default Value                                                | `parent of mysqld installation directory` |

后续略

### 5.1.8 Server System Variables

* To see the values that a server uses based on its compiled-in defaults and any option files that it reads, use this command:

```
mysqld --verbose --help
```

* To see the values that a server uses based only on its compiled-in defaults, ignoring the settings in any option files, use this command:

```mysql
mysqld --no-defaults --verbose --help
```

* To see the current values used by a running server, use the [`SHOW VARIABLES`](https://dev.mysql.com/doc/refman/8.0/en/show-variables.html) statement or the Performance Schema system variable tables. See [Section 27.12.14, “Performance Schema System Variable Tables”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variable-tables.html).

后续略

### 5.1.9 Using System Variables

```mysql
mysqld --innodb-log-file-size=16M --max-allowed-packet=1G
```

Within an option file, those variables are set like this:

```mysql
[mysqld]
innodb_log_file_size=16M
max_allowed_packet=1G
```

​	The lettercase of suffix letters does not matter; `16M` and `16m` are equivalent, as are `1G` and `1g`.

* Set a global system variable:

```mysql
SET GLOBAL max_connections = 1000;
SET @@GLOBAL.max_connections = 1000;
```

* Persist a global system variable to the `mysqld-auto.cnf` file (and set the runtime value):

```mysql
SET PERSIST max_connections = 1000;
SET @@PERSIST.max_connections = 1000;
```

* Persist a global system variable to the `mysqld-auto.cnf` file (without setting the runtime value):

```mysql
SET PERSIST_ONLY back_log = 1000;
SET @@PERSIST_ONLY.back_log = 1000;
```

* Set a session system variable:

```mysql
SET SESSION sql_mode = 'TRADITIONAL';
SET @@SESSION.sql_mode = 'TRADITIONAL';
SET @@sql_mode = 'TRADITIONAL';
```

```mysql
shell> mysql --max_allowed_packet=16M
shell> mysql --max_allowed_packet=16*1024*1024
```

With a [`LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like) clause, the statement displays only those variables that match the pattern. To obtain a specific variable name, use a [`LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like) clause as shown:

```mysql
SHOW VARIABLES LIKE 'max_join_size';
SHOW SESSION VARIABLES LIKE 'max_join_size';
SHOW VARIABLES LIKE '%size%';
SHOW GLOBAL VARIABLES LIKE '%size%';
```

