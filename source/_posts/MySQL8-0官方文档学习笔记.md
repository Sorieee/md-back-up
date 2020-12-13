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



