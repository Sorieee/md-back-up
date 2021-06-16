---
title: 深入浅出MySQL学习笔记
date: 2020-09-15 21:39:30
tags: [MySQL, 数据库]
---

# 第一部分 基础篇

# 第1章 MySQL的安装与配置

## 1.1 MySQL的下载

### 1.1.1 在Window平台下下载MySQL

 直接搜索Mysql官网下载即可。

### 1.1.2 在Linux平台下下载Mysql

#### 1.通过网页直接下载

 在网页下载后用FTP等工具传送到Linux服务器上。

#### 2.通过命令行方式下载

 wget + url的形式。

## 1.2 MySQL的安装

### 1.2.1 在Wiindows平台下安装MySQL

 略

### 1.2.2 在Linux平台下安装MySQL

 也略

## 1.3 MySQL的配置

### 1.3.1 Windows平台下配置MySQL

 略

### 1.3.2 Linux平台下配置MySQL

 通过修改cnf文件。

## 1.4 启动和关闭MySQL服务

### 1.4.1 在Windows平台下启动和关闭MySQL服务

 略

### 1.4.2 在Linux平台下启动和关闭MySQL服务

#### 1.命令行方式

- 启动服务

  ```
  cd /user/bing
  ./mysqld_safe &
  ```

- 关闭服务

  ```
  mysqladmin -uroot shutdown
  ```

#### 2.服务的方式

如果mysql是用RPM包安装的

- 启动服务

  ```
  service mysql start
  ```

- 关闭服务

```
service mysql stop
```

## 第2章 SQL基础

## 2.1 SQL简介

 略

## 2.2 MySQL使用入门

### 2.2.1 SQL分类

- DDL：数据定义语言，主要定义不同的数据段、数据库、表、列、索引等数据库对象。
- DML: 增删改查
- DCL: 数据库控制语句，比如grant，revoke等。

### 2.2.2 DDL语句

#### 1.创建数据库

```
mysql -uroot -p
#-u后面跟连接的数据库用户 -p表示需要输入密码
CREATE DATABASE dbname
#展示有哪些数据库
show databases
#选择数据库
use dbname
#展示数据表
show tables
```

#### 2.删除数据库

```
drop databse test1;
```

#### 3.创建表

```
create table tablename (
	col_name_1 col_type_1 constraints,
	col_name_2 col_type_2 constraints,
)

#查看表定义
desc tablename

#查看创建表的sql语句
show create table tablename;
```

#### 4.删除表

```
drop table tablename;
```

#### 5.修改表

```
#修改表类型
ALTER TABLE tablename MODIFY [COLUMN] column_ definition [FIRST | AFTER col_ name];
alter table emp modify ename varchar(20);

#增加表字段
ALTER TABLE tablename ADD [COLUMN] column_ definition [FIRST | AFTER col_ name];

#删除表字段
ALTER TABLE tablename DROP [COLUMN] col_ name;

# 字段改名
ALTER TABLE tablename CHANGE [COLUMN] old_ col_ name column_ definition [FIRST| AFTER col_ name]
alter table emp change age age1 int(4) ;
#修改字段排序顺序
#放在某个字段后面
alter table emp add birth date after ename;
#放在最前面
alter table emp modify age int(3) first;

#改表名
ALTER TABLE tablename RENAME [TO] new_tablename
```

### 2.2.3 DML语句

#### 1.插入记录

```
# 插入记录
INSERT INTO tablename (field1, field2,…, fieldn) VALUES( value1, value2,…, valuen);
# 不指定字段名称，values和字段排序一样
insert into emp values('lisa',' 2003- 02- 01',' 3000', 2);
#可以一次性插入多条记录
INSERT INTO tablename (field1, field2, …, fieldn) VALUES (record1_ value1, record1_ value2, …, record1_ valuesn), 
(record2_ value1, record2_ value2, …, valuesn2);
```

#### 2. 更新语句

```
UPDATE tablename SET field1= value1, field2.= value2,…, fieldn= valuen [WHERE CONDITION]
```

#### 3.删除记录

```
DELETE FROM tablename [WHERE CONDITION]
```

#### 4.查询记录

这里只介绍最基本的语法

```
SELECT * FROM tablename [WHERE CONDITION]
#查询不重复记录
select distinct deptno from emp;
#条件查询
select * from emp where deptno= 1;
#多字段条件查询
select * from emp where deptno= 1 and sal< 3000;
#排序和限制
SELECT * FROM tablename [WHERE CONDITION] [ORDER BY field1 [DESC| ASC]， field2 [DESC| ASC],…, fieldn [DESC| ASC]]
select * from emp order by sal;
#限制一部分
SELECT …[LIMIT offset_ start, row_ count]
#前三条
select * from emp order by sal limit 3;
#第二条记录开始的3条
select * from emp order by sal limit 1, 3;

#聚合
#聚合常用的有sum count max min
#group by是聚合字段
#WITH ROLLUP是可选语法，表明是否对分类聚合后的结果进行再汇总
#HAVING表示对分类后的结果再进行条件的过滤
#建议先用where过滤在用having过滤，可以提高效率
SELECT [field1, field2,…, fieldn] fun_ name FROM tablename [WHERE where_ contition] [GROUP BY field1, field2,…, fieldn [WITH ROLLUP]] [HAVING where_ contition]

#表连接
#分为内连接和外连接
#内连接仅选出两张表相互匹配的字段
#外连接会选出其他不匹配的记录
#常用内连接
select ename, deptname from emp, dept where emp. deptno= dept. deptno;
# 左连接和右连接
select ename, deptname from emp left join dept on emp. deptno= dept. deptno;
select ename, deptname from dept right join emp on dept. deptno= emp. deptno;

# 子查询
select * from emp where deptno in( select deptno from dept);
#如果子查询记录数唯一可以用=代替
select * from emp where deptno = (select deptno from dept);
#某些情况自连接可以转化为表连接
#mysql4.1之前不支持子查询
#表连接在很多情况下用于优化子查询
select emp.* from emp ,dept where emp. deptno= dept. deptno;

#记录联合
# UNION是取并集去重
# UNION ALL是不去重
SELECT * FROM t1
UNION| UNION ALL
SELECT * FROM t2 
…
UNION| UNION ALL 
SELECT * FROM tn;
```

### 2.2.4 DCL语句

 DBA用来管理系统中的对象权限时使用

```
# 创建一个用户z1，具有对sakila数据库中所有表的SELECT/INSERT权限
grant select, insert on sakila.* to 'z1'@' localhost' identified by '123';
# 回收insert权限
revoke insert on sakila.* from 'z1'@' localhost';
```

## 2.3 帮助的使用

 使用MySQL安装后自带的帮助文档

### 2.3.1 按照层次看帮助

```
# 不知道帮助能够提供什么 以下命令显示所有可供查询的分类
? contents
# 对于列出的分类， 使用"? 类别名称" 方式进一步查看
# 可以使用? 一层层往下查询
```

### 2.3.2 快速查阅帮助

```
# 如果需要快速查阅某项语法，比如show命令
? show
```

### 2.3.3 常用网络资源

```
http:// dev. mysql. com/ downloads 是 MySQL 的 官方 网 站， 可以 下载 到 各个 版本 的 MySQL 以及 相关 客户 端 开发 工具 等。
http:// dev. mysql. com/ doc 提供 了 目前 最 权威 的 MySQL 数据库 及 工具 的 在 线 手册。 
http:// bugs. mysql. com 这里 可以 查看 到 MySQL 已经 发布 的 bug 列表， 或者 向 MySQL 提交 bug 报告。 
http:// www. mysql. com/ news- and- events/ newsletter 通常 会 发布 各种 关于 MySQL 的 最新消息。
```

## 2.4 查询元数据信息

 mysql5.0以后，提供了一个新的数据库information_schema，用来记录MySQL中元数据信息。比如表名，列名。

 它是一个虚拟库，物理上不存在相关的目录和文件。库里show tables 显示的各种表也并不是实际存在的物理表，而全部是视图。

```
# 删除 数据库 test1 下 所有 前缀 为 tmp 的 表； 
select concat(' drop table test1.', table_ name,';') from tables where table_ schema=' test1' and table_ name like 'tmp%'; 

#将 数据库 test1 下 所有 存储 引擎 为 myisam 的 表 改为 innodb。
select concat(' alter table test1.', table_ name,' engine= innodb;') from tables where table_ schema=' test1' and engine=' MyISAM';
```

# 第3章 MySQL 支持的数据类型

## 3.1 数值类型

[![img](https://pic.imgdb.cn/item/5f45218e160a154a677a2b8f.jpg)](https://pic.imgdb.cn/item/5f45218e160a154a677a2b8f.jpg)

 包含严格数值类型

 INTEGER,SMALLINT,DECIMAL,NUMERIC

 以及近似数值数据类型

 FLOAT, REAL, DOUBLE PRECISION

 并在此基础上做了扩展

 TINYINT,MEDIUMINT,BIGINT

 对于整数类型，还支持在类型后面的小括号内指定显示宽度。int默认为int(11)。可以配合zerofill使用，数字位数不够填充前置0。

 整数类型有一个可选属性 UNSIGNED，保存非负数或者较大上限值，可以用此选项。取值是正常值的下限取0，上限取原值的2倍。

 整数类型还有一个属性 AUTO_INCREMENT，插入NULL到AUTO_INCREMENT列时，插入一个比该列当前最大值大1的值。

 对于任何想要使用AUTO_INCREMENT的列，应该定义为NOT NULL，并定义为PRIMARY KEY或定义为UNIQUE键。

```
CREATE TABLE AI (ID INT AUTO_ INCREMENT NOT NULL PRIMARY KEY); 
CREATE TABLE AI( ID INT AUTO_ INCREMENT NOT NULL ,PRIMARY KEY( ID));
CREATE TABLE AI (ID INT AUTO_ INCREMENT NOT NULL ,UNIQUE( ID));
```

 对于小数，有浮点数和定点数。浮点数包括float和double，定点数则只有decimal一种，定点数在MySQL内部以字符串形式存放，比浮点数精确。浮点数和定点数都可以在类型名称后面加(M,D)表示M位数字，其中D位小数位于小数点后面。

 浮点数跟(M,D)是非标准用法，最好不用这么使用。

 decimal默认10位，默认小数位为0。

 BIT(位)类型，用于存放字段值，BIT可以用来存放多位二进制数，M范围从1~64,不写默认1位。直接使用SELECT 不会看到结果，可以使用bin()或者hex()查看二进制数或者16进制结果。

## 3.2 日期时间类型

[![img](https://pic.imgdb.cn/item/5f452158160a154a6779f8a9.jpg)](https://pic.imgdb.cn/item/5f452158160a154a6779f8a9.jpg)

 如果经常插入或更新日期为当前系统时间，通常使用TIMESTAMP来表示，它返回后显示为”YYYY-MM-DD HH:MM:SS”格式字符串，显示固定宽度为19个字符。如果想获得数字值，应该在列上”+0”

 如果只是表示年份，可以用YEAR来表示，比DATE占用更少空间。有2位或者4位，4位允许值是1901~2155和0000。2位允许的是70到69，表示从1970到2069。

 从5.5.27,2为格式的year不被支持。

 零值显示方式

[![img](https://pic.imgdb.cn/item/5f45227e160a154a677b0767.jpg)](https://pic.imgdb.cn/item/5f45227e160a154a677b0767.jpg)



 可以用now()函数插入当前日期。

 创建测试表t, id1为TIMESTAMP类型

```
# create table t (id1 timestamp);
```

 会发现，系统给tm自动创建了默认值CURRENT_TIMESTAMP,并且设置了not null和 on update CURRENT_TIMESTAMP属性。

 插入NULL值时，会自动插入系统日期。MySQL只给表的第一个TIMESTAMP字段设置默认值为系统日期，如果有第二个，设置默认值为0值。

 MySQL5.6之前，可以修改id2的日期为其他常量日期，但是不能再修改为current_timestamp, 之后的版本这个限制已经去掉。

 如果将explicit_defaults_for_timestamp设置为on，那么刚才哪些默认值和属性都不会设置，需要手动操作。

```
# 查看当前时区 如果值为'SYSTEM'就和主机的时区一样
show variables like 'time_zone';
# 修改为东9区
set time_zone = '+9:00'
```

- TIMESTAMP支持的时间范围较小，DATETIME的范围更大。两者都可以设置 ON UPDATE CURRENT_TIMESTAMP属性，使得日期列可以随其他列的更新而自动更新为最新的时间
- TIMESTAMP在5.6.6版本只有添加了控制参数explicit_defauts_for_timestamp，如果为on，timestamp需要显式指定默认值和ON UPDATE CURRENT_TIMESTAMP，并且设置为not null。8.0.2版本之后默认为on，之前为off。
- explicit_defauts_for_timestamp为off，第一个TIMESTAMP列自动设置为系统时间，如果插入NULL，自动设置为当前的日期和时间，在插入或更新一行， 没有显式给定值，也会设置为当前日期和时间。如果查出取值范围，会认为该值溢出，使用0值进行填补。
- TIMESTAMP的插入和查询都受当地时区的影响，更能反映出实际的日期，而DATETIME则只能反映出插入时地区的当地的时区，其他时区的人查看数据必然会有误差。
- TIMESTAMP属性受版本和服务器SQLMode的影响很大，本章都是以MySQL5.7为例进行介绍。

 插入日期的支持的格式很多，以DATETIME为例：

- YYYY-MM-DD HH:MM:SS或YY-MM-DD HH:MM:SS的字符串，允许不严格语法，任何标点都可以用作日期部门或时间部分之间的分隔符，如果日和月小于10，不需要指定两位数。时分秒小于10也不需要指定两位数。
- YYYYMMDDHHMMSS或YYMMDDHHMMSS格式的字符串，假定字符串对日期类型是有意义的,就比较严格。数字值应为6、8、12或14位长，8或者14位，则假定为YYYYMMDD或YYYYMMDDHHMMSS。如果是6位或者12位，则认为是YYMMDD或YYMMDDHHMMSS。其他数字被解释仿佛用零填充到了最近的长度。

## 3.3 字符串类型

[![img](https://pic.imgdb.cn/item/5f464f3d160a154a67890a8d.jpg)](https://pic.imgdb.cn/item/5f464f3d160a154a67890a8d.jpg)

### 3.3.1 CHAR和VARCHAR型

 两者存储方式不同，在检索时CHAR删除了尾部空格,VARCHAR保留

### 3.3.2 BINARY和VARBINARY类型

 类似于CHAR和VARCHAR型，不同的是它们包含二进制字符串，而不包含非二进制字符串。

```
CREATE TABLE t (c BINARY( 3));
INSERT INTO t SET c=' a';
select *,hex( c), c=' a', c=' a\ 0', c=' a\ 0\ 0' from t;

最终结果是
a | 610000 | 0 | 0| 1
在保存BINARY值时，在值的最后填充'0x00'(零字节)以达到指定字符长度。
```

对比

https://www.cnblogs.com/zejin2008/p/6606120.html

[![img](https://pic.imgdb.cn/item/5f4653cc160a154a678b9c5a.jpg)](https://pic.imgdb.cn/item/5f4653cc160a154a678b9c5a.jpg)

[![img](https://pic.imgdb.cn/item/5f4653da160a154a678ba3cb.jpg)](https://pic.imgdb.cn/item/5f4653da160a154a678ba3cb.jpg)

### 3.3.3 ENUM类型

 枚举类型，值范围在创建表时，需要显示指定，1~255个成员需要1个字节存储，255到65536个成员需要两个字节。



```
create table t (gender enum('M','F'));
INSERT INTO t VALUES(' M'),(' 1'),(' f'),( NULL);
select * from t;
| M|
| M | 
| F|
| NULL |
```

 enum忽略大小写，如果不在取值范围内，默认为第一个值。

### 3.3.4 SET类型

 和ENUM类型类似，也是一个字符串对象，可以包含0~64成员

- 1~8成员 1字节
- 9~16 2字节
- 17~24 3字节
- 25~32 4字节
- 33~64 8字节

 区别在于SET类型可以一次选取多个成员

```
Create table t (col set（' a',' b',' c',' d'）; 
insert into t values(' a, b'),(' a, d, a'),(' a, b'),(' a, c'),(' a');
select * from t; 
+------+ 
| col | 
+------+ 
| a, b | 
| a, d | 
| a, b | 
| a, c | 
| a |
```

 只允许插入允许的值，多个值合并为一个值。

## 3.4 JSON类型

 从5.7.8之后支持

- 插入时会自动校验数据是否json格式，不是就报错
- 提供了相关的内置函数吗，可以方便地提取各类数据，可以修改特定的值
- 优化存储格式，存储在json格式中的数据会被转换成内部的存储格式，允许快速读取。

 支持的数据类型包括NUMBER、STRING、BOOLEAN、NULL、ARRAYA、OBJECT 6种。

```
["abc", 10, null, true, false]
{"k1": "value", "k2": 10}
[99, {"id": "hk500", "cost": 75.99}]
```

常见null,true,false必须小写才合法，’null’, ‘false’合法, ‘NULL’或者’FALSE’不合法。

如果要包含单引号或者双引号，需要反斜线转义。

json不可有默认值，最大长度取决于max_allowed_packet(存储时限制，内存中计算允许超过该值)。

# 第4章 MySQL中的运算符

## 4.1 算术运算符

[![img](https://pic.imgdb.cn/item/5f465937160a154a678e5f3d.jpg)](https://pic.imgdb.cn/item/5f465937160a154a678e5f3d.jpg)

 在除法和模运算中，如果除数为0，返回结果为NULL。

 模运算可以用MOD(a,b)函数。

## 4.2 比较运算符

[![img](https://pic.imgdb.cn/item/5f4659ce160a154a678eb77e.jpg)](https://pic.imgdb.cn/item/5f4659ce160a154a678eb77e.jpg)

 注意：NULL不能用=比较

## 4.3 逻辑运算符

[![img](https://pic.imgdb.cn/item/5f465a5e160a154a678f074f.jpg)](https://pic.imgdb.cn/item/5f465a5e160a154a678f074f.jpg)

- 逻辑非，当操作数为0返回1,1返回0，NULL返回NULL
- 逻辑与，操作数有一个为NULL就为NULL
- 逻辑或，两个全为NULL，或者有一个为NULL其他全是0，结果为NULL。
- 逻辑异或，有NULL则返回NULL。

## 4.4 位运算符

[![img](https://pic.imgdb.cn/item/5f465c78160a154a67901409.jpg)](https://pic.imgdb.cn/item/5f465c78160a154a67901409.jpg)

 是将给定的操作数转换成二进制进行计算

## 4.5 运算符优先级

[![img](https://pic.imgdb.cn/item/5f465d5b160a154a67908550.jpg)](https://pic.imgdb.cn/item/5f465d5b160a154a67908550.jpg)

# 第5章 常用函数

## 5.1 字符串函数

[![img](https://pic.imgdb.cn/item/5f47a459160a154a6766008e.jpg)](https://pic.imgdb.cn/item/5f47a459160a154a6766008e.jpg)

[![img](https://pic.imgdb.cn/item/5f47a466160a154a67660473.jpg)](https://pic.imgdb.cn/item/5f47a466160a154a67660473.jpg)

## 5.2 数值函数

[![img](https://pic.imgdb.cn/item/5f47a4a1160a154a676616a5.jpg)](https://pic.imgdb.cn/item/5f47a4a1160a154a676616a5.jpg)

## 5.3 日期和时间函数

[![img](https://pic.imgdb.cn/item/5f47a4ef160a154a67662cf9.jpg)](https://pic.imgdb.cn/item/5f47a4ef160a154a67662cf9.jpg)

DATE_FORMAT的fmt的时间格式。

[![img](https://pic.imgdb.cn/item/5f47a583160a154a676659af.jpg)](https://pic.imgdb.cn/item/5f47a583160a154a676659af.jpg)

 DATE_ADD(date, INTERVAL exptr type)函数。其中INTERVAL是间隔类型关键字，expr是一个表达式，type是间隔类型。

 MySQL的日期间隔类型

[![img](https://pic.imgdb.cn/item/5f47a696160a154a6766ae6a.jpg)](https://pic.imgdb.cn/item/5f47a696160a154a6766ae6a.jpg)

```
select now(), date_add(now(), INTERVAL 31 day), date_add(now(), INTERVAL '1_2' year_month);
```

## 5.4 流程函数

[![img](https://pic.imgdb.cn/item/5f47a757160a154a6766e93c.jpg)](https://pic.imgdb.cn/item/5f47a757160a154a6766e93c.jpg)

## 5.5 JSON函数

[![img](https://pic.imgdb.cn/item/5f47a8a4160a154a6767537b.jpg)](https://pic.imgdb.cn/item/5f47a8a4160a154a6767537b.jpg)

[![img](https://pic.imgdb.cn/item/5f47a82b160a154a67672ee8.jpg)](https://pic.imgdb.cn/item/5f47a82b160a154a67672ee8.jpg)

## 5.6 窗口函数

| 函数           | 功能                             |
| :------------- | :------------------------------- |
| ROW_NUMBER()   | 分区中的行号                     |
| RANK()         | 当前行在分区中的排名，含序号间隙 |
| DEMSE_RANK()   | 当前行在分区中的排名，无序号间隙 |
| PERCENT_RANK() | 百分比等级值                     |
| CUME_DIST()    | 累计分配值                       |
| FIRST_VALUE()  | 窗口中第一行的参数值             |
| LAST_VALUE()   | 窗口中最后一行的参数值           |
| LAG()          | 分区中指定行落后于当前行的参数值 |
| LEAD()         | 分区中领先当前行的参数值         |
| NTH_VALUE()    | 从第N行窗口框架的参数值          |
| NTILE()        | 分区中当前行的桶号               |

## 5.7 其他常用函数

[![img](https://pic.imgdb.cn/item/5f47aa85160a154a6767d6b2.jpg)](https://pic.imgdb.cn/item/5f47aa85160a154a6767d6b2.jpg)

# 第二部分 开发篇

# 第6章 表类型(存储引擎)的选择

## 6.1 MySQL存储引擎概述

 5.5版本之前默认是MyISAM，之后是InnoDB。

 通过修改参数文件的default_storage_engine调整引擎。

 MySQL5.7支持的引擎有InnoDB，MyISAM，MEMORY, CSV, BLACKHOLE, ARCHIVE, MERGE(MRG_MyISAM), FEDERATED, EXAMPLE, NDB等。

```
# 查看当前默认引擎
show variables 'default_storage_engine'
# 查询当前数据库支持的存储引擎
show engines 
# 修改某个表的存储引擎
atler table ai engine = innodb
```

## 6.2 各种存储引擎的特性

[![img](https://pic.imgdb.cn/item/5f47adca160a154a6768d9ba.jpg)](https://pic.imgdb.cn/item/5f47adca160a154a6768d9ba.jpg)

### 6.2.1 MyISAM

 不支持事务、外键。某些场景相对InnoDB访问速度有明显优势。对事务完整性没有要求或者以SELECT、INSERT为主的表可以使用这个引擎来创建表。

 在磁盘上存储为三个文件，文件名与表名都相同

- .frm(存储表定义)
- .MYD(MYData, 存储数据)
- .MYI(MyIndex，存储索引)

 数据文件和索引文件可以防止在不同的目录，平均分布IO，获得更快速度。

 要指定索引文件和数据文件路径，在创建表时，通过DATA DIRECTORY和INDEX DIRECTORY语句指定。路径需要绝对路径，并且具有访问权限。

 表可能损坏，损坏后不可访问，需要按提示需要球服或返回访问错误的结果。提供修复工具，可以用CHECK TABLE语句来检查MyISAM表的健康，并用 REPAIR TABLE 语句修复一个损坏的表。

 表支持三种不同格式

- 静态(固定长度)表
- 动态表
- 压缩表

 其中，静态表是默认存储格式，字段都是非边长字段，每个记录都是固定长度。有点事存储迅速，容易缓存，故障易修复，缺点是占用空间更大。会填补空格，并且应用之前去掉空格，如果结尾存储了有空格的数据，应用时会被去掉。

### 6.2.2 InnoDB

 具有提交、回滚和崩溃恢复能力的事务安全保障，提供了更小的锁粒度和更强的并发能力。

 会比MyISAM占用更多的磁盘以保留数据和索引。

#### 1.自动增长列

 自动增长列可以手动插入，但是如果为空，插入的值是自动增长的值。

 MySQL8之前，重启后，会将自增列设置为当前存储的最大值+1

 之后得到了修复，实现方式是将自增逐渐的计数器持久化到REDO LOG中，每次计数器发生变化吗，都会将其写入到REDO LOG。重启后，从REDO LOG取值。

 可以使用LAST_INSERT_ID()查询最后插入记录使用的值，如果人为指定，那么LASTE_INSERT_ID()不会更新。

#### 2.外键约束

 只有innodb支持外键，创建外键，要求父表必须有对应索引。子表创建时，也会自动创建对应的索引。

 在创建索引时，可以指定删除、更新父表，对字表进行的想要操作，包括RESTRICT、CASCADE, SET NULL和NO ACTION。其中RESTRICT和NO ACTION相同，是指限制字表有关联记录的情况下父表不能更新。

 CASCADE在父表进行更新或删除时，更新或删除字表对应记录；

 SET_NULL 代表父表在更新或删除时，字表对应的字段被设置为NULL。

 选择后两种方式要谨慎，可能会因为错误的操作导致数据的丢失。

 如果导入多个表的数据，可以暂时先忽略外键检查，SET FOREIGN_KEY_CHECKS = 0,执行完成后，执行 SET FOREIGN_KEY_CHECKS = 1恢复原状态。

#### 3.主键和索引

 InnoDB的数据文件本身就是以聚簇索引的形式保存的，这个聚簇索引也被称为主索引，而且也是表的主键，表的每行数据保存在主索引的叶子节点上。因此，所有表都必须包含主键。如果没有显示指定主键，会自动创建一个长度为6个字节的long类型的隐藏字段作为主键。

 主键选择原则：

- 满足唯一和非空约束
- 优先考虑使用最经常被当作查询条件的字段或自增字段
- 字段值基本不会被修改
- 尽可能使用短的字段。

 除主键外，其他索引都叫作辅助索引或者二级索引。二级索引会指向主索引，并通过主索引获取最终的数据。

#### 4.存储方式

 两种：

- 使用共享表空间存储：表结构放在.frm文件中，数据和索引保存在inno_data_home_dir和inno_data_file_path定义的表空间中，可以使多个文件。
- 使用多表空间存储：这种方式创建的表的表结构仍然保存在.frm文件中，但是每个表的数据和索引单独保存在.ibd中。如果是一个分区表，则每个分区对应单独的.ibd文件，文件名是“表名+分区名”，可以在创建分区时指定每个分区的数据文件的位置，以此分布IO在不同的多个磁盘上。

 共享表空间时，随着数据增长，表空间维护会愈来愈困难。所以都建议使用多表空间。设置参数innodb_file_per_table，默认设置为ON。

### 6.2.3 MEMORY

 只对应一个文件.frm，索引默认使用HASH索引，一旦服务关闭，数据丢失。

 在启动时选择使用的–init-file选项， 可以在服务启动时就从持久稳固的数据源装载表。

 需要足够内存来维持所有在同一时间使用的MEMORY表，不需要就删除。

 表大小收到max_heap_table_size系统变量约束，初始值是16MB，创建表通过MAX_ROWS指定表的最大行数。

 主要用于变化不频繁的代码表，或者作为统计操作中间表。便于高效地对中间结果进行分析并得到最终的统计结果。

### 6.2.4 MERGE

 暂略

### 6.2.5 TokuDB

 略

## 6.3 如何选择合适的存储引擎

 暂略

# 第7章 选择合适的数据类型



## 7.1 CHAR和VARCHAR

[![img](https://pic.imgdb.cn/item/5f49a6b7160a154a67e55484.jpg)](https://pic.imgdb.cn/item/5f49a6b7160a154a67e55484.jpg)

 最后一行的值只用于MySQL，如果非严格模式，超过列长度的值将不会保存。如果是严格模式，则会提示错误。

 CHAR是固定长度，所以处理速度会快很多，但是随着MySQL不断升级，VARCHAR类型的性能也在不断改进和提高，所以VARCHAR类型被更多地使用。

 不同引擎使用的原则有所不同。

 MyISAM：建议使用CHAR

 MEMORY：都使用固定长度的数据航，所以没区别，都当成CHAR处理。

 InnoDB：建议使用VARCHAR。内部行存储格式没有区别固定长度和可变长度，所以固定长度不一定比使用可变长度VARCHAR列性能要好。

## 7.2 TEXT和BLOB

 BLOB能用来保存二进制数据，比如照片，TEXT只能保存字符数据，比如一篇文章或者日记。

 TEXT和BLOB又分为TINYTEXT,TEXT,MEDIUMTEXT,LONGTEXT

 和TINYBLOB,BLOB,MEDIUMBLOB,LONGBLOB

 1.两者删除会引起一些性能问题，特别是执行大量地删除操作的时候

 会在数据表留下很大的空洞，以后填入这些空洞的记录在插入的性能上会有影响。为了提高性能，建议定期使用

 OPTIMIZE TABLE功能对这类表进行碎片整理。

 对于InnoDB，OPTIMIZE语句被优化为recreate+analyze语句。

 2.可以使用合成的索引大大提高大文本的查询性能。

 简单地说就是根据大文本的内容建立一个散列值，并把这个值存储在单独的数据列中，接下来就可以通过搜索散列值找到数据行了，只能用于精确匹配。

 MySQL提供了前缀索引的功能，就是只为字段的前n列创建索引。

```
create index idx_blob on t(conext(100));
```

 查询时，%不能放在最前面，否则索引不可用、

 3.在不必要时避免检索大型的BLOB和TEXT值。

 例如 SELECT * 就不是很好的想法，除非能够确定WHERE子句只会找到所需要的行。

 4.把BLOB和TEXT列分离到单独的表中

 这会减少主表的碎片，显著减少主表的数据列从而获得性能优势。

## 7.3 浮点数与定点数

 浮点数在插入时如果超过了该列定义的实际精度，会被四舍五入到实际定义的值。float和double(或real)来表示浮点数。

 定点数不同于浮点数，实际上是以字符串形式存放的，可以更精确地表示数据。如果实际插入的精度大于定义的精度，会进行警告（默认的SQLMode下），但是数据库按照实际地精度四舍五入后插入；如果SQLMode是TRADITIONAL，则系统会保存。decimal(或numberic)用来表示定点数。

## 7.4日期类型选择

- 如果只存年，那么就用YEAR类型，可以提高效率
- 如果需要记录年月日时分秒，并且记录的年份比较久远，最少使用DATETIME
- 如果记录的日期需要让不同时区使用，最好使用TIMESTAMP

# 第8章 字符集

## 8.1 字符集概述

 字符集就是一套文字符号及其编码、比较规则的集合。

## 8.2 Unicode简述

 为了统一字符编码，国际标准组织在1984年创造了UCS，标准编号则定位了ISO-10646，采用4字节(32bit)编码，因此简称UCS-4。

 具体规则是将代码空间分为组，面，行，格。1到4字节分别代表组，面，行，格。并且规定第32位必须为0，每个面的最后两个码位FFFEh和FFFFh保留不用。因此共有128个群组，每个群组有256个面，每个面有256行，每行包括256格。发布后就遭到了部分美国公司反对。

 1988年，Xerox公司提议指定新的以16位编码的统一字符集Unicode，并且联合各大公司成立Unicode协会，并成立Unicode技术委员会，专门负责Unicode文字收集、整理和编码，并于1991年推出了Unicode1.0。

 两套编码不利，1991年10月达成协议，Unicode并入ISO-10646的0组0面。称之为基本多语言字面(BMP)。

 大部分用户使用BMP就够用了。早期ISO标准只要求实现BMP字面，那么只需要两个字节来编码就够了，Unicode也就是这么做的，称为UCS-2。

 绝大部分情况下，Unicode都能满足要求，而且在节省内存和处理时间上都有又是。

 如果要使用BMP以外的文字怎么办，Unicode提出了名为UTF-16或代理法的解决方案。对BMP编码保持两字节不变，对其他字面的文字按一定规则将32位转换成两个16位的Unicode编码。

## 8.3 汉字常见的一些字符集

- GB 2312-80: 6763个汉字和683个非汉字图形符号
- GB 13000: 27484个汉字以及一些偏旁部首等，但是没有业界支持，形式上的标准。
- GBK
- GB 18030: 采用2或4字节编码，2字节部分与GBK一致，是GBK的超集。

## 8.4 选择合适的字符集

 Mysql5.7支持几十种字符集包括UCS-2, UTF-16, UFT-8, UTF-16LE, UTF-32等字符集。

1. 满足不同国家和地区的需求，应该选择Unicode字符集，最常用的就是UTF-8。更严谨的说法是编码规则是UTF-8， 其中utf8mb3和utf8mb4是最常用的两个字符集，MySQL8.0，默认字符集已经又latin1变为utf8mb4。
2. 如果涉及到已有数据导入，就要充分考虑数据库字符集对已有数据的兼容性。
3. 数据库只需要支持一般中文，而且数据量很大，性能要求也很高，可以使用GBK。如果应用主要处理英文字符，仅有少量汉字数据，那么UTF-8更好。
4. 如果数据库需要做大量的字符运算，比如比较、排序，定长字符集更好。
5. 如果所有客户端都支持相同的字符集，应该优先选择该字符集作为数据字符集。

## 8.5 MySQL支持的字符集简介

 支持多种字符集，同一台服务器、同一个数据库、甚至同一张表的不同字段都可以指定使用不同的字符集。

 查看所有字符集的命令是show character set;

 或者查看information_schema.character_set，可以显示所有的字符集和该字符集默认的排序规则。

 MySQL字符集包括字符集和排序规则两个概念，字符集和排序规则是一对多的概念，支持30多种字符集和70多种排序规则。

 每个字符集至少对应一个排序规则，可以使用SHOW COLLATION LIKE ‘***’命令或者通过系统表information_schema.COLLATIONS查看相关字符集的排序规则。

## 8.6 MySQL数据集的设置

### 8.6.1 服务器字符集和排序规则

- 在my.conf设置

  ```
  character-set-server=utf8
  ```

- 在启动项中指定

```
mysqld --character-set-server=utf8
```

- 或者在编译时指定

```
cmake . -DDEFAULT-CHARSET=utf8
```

可以使用show variables like ‘character_set_server’

show variables like ‘collation_server’

查看当前服务器字符集和排序规则。

### 8.6.2 数据库字符集和排序规则

- 如果指定了字符集和排序规则，则使用指定的字符集和排序规则
- 如果都没有指定，则使用默认
- 如果只指定了字符集，则使用该字符的默认排序规则
- 如果只指定的排序规则，使用该规则对应的字符集

 推荐在创建数据库是明确指定字符集和排序规则，避免受到默认值影响。

 显示当前数据库的字符集和排序规则

```
show variables like 'character_set_database'

show variables like 'collation_database'
```

### 8.6.3 表字符集和排序规则

创建表时，添加如下语句

[![img](https://pic.imgdb.cn/item/5f49b76a160a154a67e9a32e.jpg)](https://pic.imgdb.cn/item/5f49b76a160a154a67e9a32e.jpg)

### 8.6.4 列字符集和校对规则

 一般不设置，只是MySQL提供的灵活设置手段，可以在创建表时指定或者修改表时调整。

### 8.6.5 连接字符集和校对规则

 以上4中设置方式，确定的是数据保存的字符集和校对隔着，对于实际应用访问，还存在客户端与服务器之间交互的字符集和校对规则的设置。

 提供了3个不同的参数

 character_set_client

 character_set_connection

 character_set_results。

 代表客户端、连接和返回结果的字符集。

 通常应该相同，才可以确保用户写入数据的正确读出。

## 8.7 字符集的修改步骤

 如果再开始阶段没有正确设置字符集，需要调整，不能直接通过

 alter database character set ***或者

 alter table character set ***进行。

 这两个命令都没有更新已有记录的字符集

 以下是模拟latin1字符集修改成GBK字符集的过程

 1.导出表结构

 mysqldump -uroot -p –default-character-set=gbk -d databasename>createtable.sql

 -d代表只导出表结构不导出数据

 2.手工修改createtable.sql中表结构定义中的字符集为新的字符集。

 3.确保记录不再更新，导出所有记录

```
mysqldump -uroot -p --quick --no- create- info --extended- insert --default- character- set= latin1 databasename> data. sql
```

[![img](https://pic.imgdb.cn/item/5f49ba1d160a154a67ea5a4c.jpg)](https://pic.imgdb.cn/item/5f49ba1d160a154a67ea5a4c.jpg)

4.打开data.sql， 将SET NAMES latin1修改为 SET NAMES gbk

5.使用新的数据集创建新的数据库

```
create database databasename default charset gbk;
```

6.创建表，执行createtab.sql

```
mysql -uroot -p databasename < createtab. sql
```

7.导入数据，执行data.sql

```
mysql -uroot -p databasename < data. sql
```

# 第9章 索引的设计和使用

## 9.1 索引概述

 根据引擎可以定义每个表的最大索引数和最大索引长度，每种引擎对每个表至少支持16个索引，总索引长度至少为256字节。大多数存储索引有更高的限制。

 MyISAM和InnoDB默认创建的都是BTREE索引。MySQL5.7以后可以通过虚拟列索引来实现函数索引的功能，通识MySQL也支持前缀索引，即对索引字段的前N个字符创建索引。

 对于MyISAM引擎，索引长度可以到达1000字节长，而InnoDB，最长是3072字节。注意，前缀的限制应该以字节为单位进行测量。CREATE TABLE语句中的前缀长度解释为字符数。

 还支持全文本索引，该索引可以用于全文搜索，MySQL5.6之后，以上两个引擎都支持FUULTEXT索引，但只限于CHAR,VARCHAR和TEXT列。索引总是对整个列进行的，不支持局部索引。

 也可以对空间列类型创建索引，MySQL5.7之前只有MyISAM支持，后面版本InnoDB也支持，索引以R-Trees的数据结构保存。

 默认情况下,MEMORY存储引擎使用HASH索引，但也支持BTREE索引。

 可以在新建表是设置索引，也可以单独添加。

[![img](https://pic.imgdb.cn/item/5f49c157160a154a67ec6d28.jpg)](https://pic.imgdb.cn/item/5f49c157160a154a67ec6d28.jpg)

 也可以通过ALTER TABLE的形式进行添加。

## 9.2 设计索引的原则

- 在条件列上创建索引
- 尽量使用唯一索引，索引值分布越大，效果越好。例如性别等只有F和M的列，不需要索引。
- 使用短索引，对字符串进行索引，应该指定一个索引长度，如果一个CHAR(200)列，如果只在前10个到20个字符内，多数值是唯一的，就不要对整个列进行索引。也会使查询更快，占用空间更小，索引高速缓存中的块能容纳更多的键值。
- 利用最左前缀。创建一个n列的索引，实际创建了看MySQL可利用的n个索引。可以起到几个索引的作用，因为可利用索引中最左边的列集来匹配行，这些列集称为最左前缀。比如创建了a,b,c的组合索引，那么a=?或a=? and b = ? 或者a=? and b = ? and c = ?都可以使用这个索引。
- 对于InnoDB引擎的表，尽量手工指定主键，因为会按主键顺序保存，如果没有，但是有唯一索引，就按唯一索引保存，如果都没有，就会生成一个内部列，按这个顺序保存。按照主键或者内部列访问最快，所以尽量指定主键。可以选择最长访问的列作为主键，而且主键尽量选择较短的数据类型的列，有效减少索引的磁盘占用，提高索引缓存效果。

## 9.3 索引设计的误区

- 不是所有表都需要创建索引，常见的代码表、配置表等数据量很小的表，除了主键，再创建表没有太大的意义。
- 不要过度索引，会给查询带来负担
- 谨慎创建低选择索引，对于选择性低并且数据分布均匀的列，索引效果不好。（MySQL8.0之后有一个直方图可以）

## 9.4 索引设计的一般步骤

- 整理表上所有SQL的WHERE条件所用到的列组合、关联查询的关联条件
- 整理所有SQL的预期执行频率
- 整理所有涉及列的选择度
- 遵照之前提到的设计原则，给表选择合适的主键
- 优先给那么执行频率最高的SQL创建索引。
- 按执行频率排序，检查是否有必要为每个SQL创建索引
- 索引和并，利用复合索引降低索引的总数
- 上线之后，通过慢查询分析、执行计划分析、索引使用统计，来确定索引的使用情况。

## 9.5 BTREE索引与HASH索引

 HASH索引有一些重要特征：

- 只用于=或者<=>操作符的等式比较
- 优化器不能使用HASH索引来加速ORDER BY等操作
- MySQL不能确定两个值之间大约有多少行。如果将一个MyISAM的表改为HASH索引的MEMORY表，会影响一些查询效率
- 只能使用整个关键字来索索一行。

 对于BTREE，当使用>, <, >=, <=, BETWEEN、!=或者<>，或者LIKE操作符时，都可以使用相关列上的索引。

## 9.6 索引在MySQL8.0中的改进

### 9.6.1 不可见索引

 可以创建索引时通过invisible设置不可见，这个就是对查询优化器不可见。

 目的是为了减少对表上索引进行调整时的潜在风险。如果删除一个大表索引，应该设置为不可见，然后看是否有很大影响，就可以很方便地恢复回去。

### 9.6.2 倒序索引

 可以创建索引时指定倒序，相对于Oracle的倒序索引对查询的优化效果，MySQL的作用还是比较弱。由于引入倒序索引group by的隐式排序的特性取消了，要注意。

# 第10章 开发常用数据库对象

## 10.1 视图

### 10.1.1 什么是视图

```
视图是一种虚拟存在的表。
```

- 简单：使用视图的用户不需要管理后面对应表的结构
- 安全：使用视图的用户只能访问他们被允许查询的结果集
- 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响；源表修改列可以通过修改视图来解决，不会造成访问者的影响

### 10.1.2 视图操作

```
包括创建、修改、删除和查看视图定义。
```

### 10.1.3 创建或修改视图

如果要用CREATE OR REPLACE 或者ALTER，还需要有DROP视图的权限。

[![img](https://pic.imgdb.cn/item/5f4af50e160a154a674d3812.jpg)](https://pic.imgdb.cn/item/5f4af50e160a154a674d3812.jpg)

[![img](https://pic.imgdb.cn/item/5f4af534160a154a674d453b.jpg)](https://pic.imgdb.cn/item/5f4af534160a154a674d453b.jpg)

```
5.7.7版本之前，FROM关键字后面不能包含子查询。

视图可更新性和视图中查询定义有关：
```

- 包含以下关键字的SQL语句，聚合函数,DISTINCT,GROUP BY, HAVING, UNION, UNION ALL
- 常量视图
- JOIN
- FROM一个不能更新的视图
- WHERE的子查询应用了FROM语句中的表。

其中WITH [CASCADED | LOCAL] CHECK OPTION决定了是否允许更新数据使记录不再满足视图的条件。

- LOCAL:只要满足本视图的条件就可以更新

- CASCADED：必须满足所有针对盖世兔的所有视图的条件才可以更新。

  默认是CASCADED。

### 10.1.4 删除视图

```
DROP VIEW [IF EXISTS] view_ name [, view_ name] . .[RESTRICT | CASCADE]
```

在使用SHOW TABLE STATUS，不仅可以显示表的信息，同时也可以显示视图的信息。

```
SHOW TABLE STATUS [FROM DB_NAME] [LIKE 'pattern']
```

使用SHOW CREATE VIEW进行视图定义查看。

```
show create view staff_list
```

最后，通过查看系统表information_schema.views也可以查看视图的相关信息。

## 10.2 存储过程和函数

 5.0版本开始支持存储过程和函数。

### 10.2.1 什么是存储过程和函数

 是事先经过编译并分出在数据库中的一段SQL语句的集合。

 存储过程和函数的却别在于函数必须有返回值。

### 10.2.2 存储过程和函数的相关操作

 创建存储过程和函数需要CREATE ROUNTE权限，修改删除需要 ALTER ROUTE权限。执行存储过程需要EXECUTE权限。

### 10.2.3 创建、修改存储过程或者函数

 创建存储过程或函数的语法如下

```
CREATE PROCEDURE sp_ name 
([proc_ parameter[,. .]])
	[characteristic . .] routine_ body CREATE FUNCTION sp_ name 
	([func_ parameter[,. .]]) 
	RETURNS type 
	[characteristic . .] routine_ body
proc_ parameter:
[ IN | OUT | INOUT ] param_ name type
func_ parameter: 
param_ name type 
type: 
Any valid MySQL data type
characteristic:
LANGUAGE SQL 
| [NOT] DETERMINISTIC
| { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA } 
| SQL SECURITY { DEFINER | INVOKER } 
| COMMENT 'string' 
routine_ body: Valid SQL procedure statement or statements
```

修改

```
ALTER {PROCEDURE | FUNCTION} sp_ name [characteristic . .] 
characteristic: 
{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
| SQL SECURITY { DEFINER | INVOKER }
| COMMENT 'string'
```

调用语法如下

```
CALL sp_name([parameter[,.  .]])
```

MySQL的存储过程和函数允许DDL语句，也允许在存储过程中执行提交或者回滚，但是存储过程和函数中不允许执行LOAD DATA INFILE语句。其中可以调用其他的存储过程或函数。

[![img](https://pic.imgdb.cn/item/5f4b998b160a154a677a8cf2.jpg)](https://pic.imgdb.cn/item/5f4b998b160a154a677a8cf2.jpg)

 通常在创建存储过程和函数之前，都会通过DELIMITER $$命令将语句的结束符从;修改为其他符号。

```
CALL film_ in_ stock( 2, 2,@ a);
SELECT @a;
```

characteristic特征值的部分进行简单的说明：

- LANGUAGE SQL: 说明下面的BODY是SQL编写的，为MySQL以后支持其他语言做准备

- [NOT] DETERMINISTIC: DETERMINISTIC确定的，即每次输入一样那么输出一样的程序，NOT DETERMINISTIC是不确定的，默认是非确定的。当前，这个值还没有被优化程序使用。

- {CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA}: 这个特征值提供子程序使用数据的内在信息，这些特征值目前知识提供给服务器。CONTAINS SQL 表示 子程序 不 包含 读 或 写 数据 的 语句。NO SQL 表示 子程序 不 包含 SQL 语句。 READS SQL DATA 表示 子程序 包含 读 数据 的 语句，但不 包含 写 数据 的 语句。 MODIFIES SQL DATA 表示 子程序 包含 写 数据 的 语句。

  如果 这些 特征 没有 明确 给定， 默认 使 用的 值 是 CONTAINS SQL。

- SQL SECURITY { DEFINER | INVOKER }： 可以 用来 指定 子程序 该 用 创建 子程序 者 的 许可 来 执行， 还是 使用 调 用者 的 许可 来 执行。 默认值 是 DEFINER。 *
- COMMENT ‘string’： 存储 过程 或者 函数 的 注释 信息。

### 10.2.4 删除存储过程或函数

[![img](https://pic.imgdb.cn/item/5f4b9e49160a154a677c186a.jpg)](https://pic.imgdb.cn/item/5f4b9e49160a154a677c186a.jpg)

### 10.2.5 查看存储过程或者函数

#### 1.查看存储过程或者函数的状态

[![img](https://pic.imgdb.cn/item/5f4b9e9b160a154a677c2e3c.jpg)](https://pic.imgdb.cn/item/5f4b9e9b160a154a677c2e3c.jpg)

#### 2.查看存储过程或函数的定义

[![img](https://pic.imgdb.cn/item/5f4b9eb0160a154a677c335b.jpg)](https://pic.imgdb.cn/item/5f4b9eb0160a154a677c335b.jpg)

#### 3.通过information_schema.Rounties了解存储过程和函数的信息

### 10.2.6 变量的使用

#### 1.变量的定义

 通过DECLARE 声明

[![img](https://pic.imgdb.cn/item/5f4ba05e160a154a677ca559.jpg)](https://pic.imgdb.cn/item/5f4ba05e160a154a677ca559.jpg)

```
DECLARE last_ month_ start DATE;
```

#### 2.变量的赋值

[![img](https://pic.imgdb.cn/item/5f4ba0c0160a154a677cc272.jpg)](https://pic.imgdb.cn/item/5f4ba0c0160a154a677cc272.jpg)

### 10.2.7 定义条件和处理

#### 1.条件的定义

[![img](https://pic.imgdb.cn/item/5f4ba10f160a154a677cd658.jpg)](https://pic.imgdb.cn/item/5f4ba10f160a154a677cd658.jpg)

#### 2.条件的处理

```
DECLARE handler_ type HANDLER FOR condition_ value[,. .] sp_ statement 
handler_ type: 
CONTINUE 
| EXIT 
| UNDO 
condition_ value: 
SQLSTATE [VALUE] sqlstate_ value 
| condition_ name 
| SQLWARNING 
| NOT FOUND
```

1.当没有条件处理时结果如下

[![img](https://pic.imgdb.cn/item/5f4ba2a5160a154a677d433d.jpg)](https://pic.imgdb.cn/item/5f4ba2a5160a154a677d433d.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba2b2160a154a677d469f.jpg)](https://pic.imgdb.cn/item/5f4ba2b2160a154a677d469f.jpg)

2.对主键冲突进行异常处理时

[![img](https://pic.imgdb.cn/item/5f4ba220160a154a677d228d.jpg)](https://pic.imgdb.cn/item/5f4ba220160a154a677d228d.jpg)

 hanlder_type现在只支持CONTINUE和EXIT两种。

 condition_value的值既可以是DECLARE定义的condition_name，也可以是SQLSTATE值或者mysql-error-code的值或者SQLWARNING, NOT FOUND, SQLEXCEPTION。这3个值是定义好的错误类别。

[![img](https://pic.imgdb.cn/item/5f4ba3c5160a154a677d954d.jpg)](https://pic.imgdb.cn/item/5f4ba3c5160a154a677d954d.jpg)

上面例子还可以写成以下几种方式：

[![img](https://pic.imgdb.cn/item/5f4ba3e6160a154a677d9dcc.jpg)](https://pic.imgdb.cn/item/5f4ba3e6160a154a677d9dcc.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba3f3160a154a677da103.jpg)](https://pic.imgdb.cn/item/5f4ba3f3160a154a677da103.jpg)

### 10.2.8 光标的使用

 在存储过程和函数中，可以使用光标对结果集进行循环的处理。

[![img](https://pic.imgdb.cn/item/5f4ba4ea160a154a677de33d.jpg)](https://pic.imgdb.cn/item/5f4ba4ea160a154a677de33d.jpg)

 以下是一个例子，对payment表按照行进行循环处理。处理， 按照 staff_ id 值 的 不同 累加 amount 的 值， 判断 循环 结束 的 条件 是 捕获 NOT FOUND 的 条件， 当 FETCH 光标 找 不到 下一 条 记录 的 时候， 就会 关闭 光标 然后 退出 过程。

[![img](https://pic.imgdb.cn/item/5f4ba5bc160a154a677e2334.jpg)](https://pic.imgdb.cn/item/5f4ba5bc160a154a677e2334.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba5dc160a154a677e2cbd.jpg)](https://pic.imgdb.cn/item/5f4ba5dc160a154a677e2cbd.jpg)

注意：变量、 条件、 处理 程序、 光标 都是 通过 DECLARE 定义 的， 它们 之间 是有 先后 顺序 要求 的。 变量 和 条件 必须 在最 前面 声明， 然后 才能 是 光标 的 声明， 最后 才 可以 是 处理 程序 的 声明。

### 10.2.9 流程控制

#### 1.IF 语句

[![img](https://pic.imgdb.cn/item/5f4ba69f160a154a677e5fd7.jpg)](https://pic.imgdb.cn/item/5f4ba69f160a154a677e5fd7.jpg)

#### 2.CASE语句

[![img](https://pic.imgdb.cn/item/5f4ba6b2160a154a677e64a3.jpg)](https://pic.imgdb.cn/item/5f4ba6b2160a154a677e64a3.jpg)

或

[![img](https://pic.imgdb.cn/item/5f4ba6c3160a154a677e6a17.jpg)](https://pic.imgdb.cn/item/5f4ba6c3160a154a677e6a17.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba6d0160a154a677e6f73.jpg)](https://pic.imgdb.cn/item/5f4ba6d0160a154a677e6f73.jpg)

#### 3.LOOP语句

[![img](https://pic.imgdb.cn/item/5f4ba6f3160a154a677e796f.jpg)](https://pic.imgdb.cn/item/5f4ba6f3160a154a677e796f.jpg)

#### 4.LEAVE语句

 从标注的流程构造中退出，通常和BEGIN和END或循环一起使用

[![img](https://pic.imgdb.cn/item/5f4ba71f160a154a677e8603.jpg)](https://pic.imgdb.cn/item/5f4ba71f160a154a677e8603.jpg)

#### 5. ITERATE语句

 必须用在循环中，作用类似于CONITNUE

[![img](https://pic.imgdb.cn/item/5f4ba783160a154a677ea36c.jpg)](https://pic.imgdb.cn/item/5f4ba783160a154a677ea36c.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba78e160a154a677ea848.jpg)](https://pic.imgdb.cn/item/5f4ba78e160a154a677ea848.jpg)

#### 6.REPEAT语句

 有条件的循环控制语句，满足条件时退出循环

[![img](https://pic.imgdb.cn/item/5f4ba7bc160a154a677eb7f4.jpg)](https://pic.imgdb.cn/item/5f4ba7bc160a154a677eb7f4.jpg)

#### 7.WHILE语句

[![img](https://pic.imgdb.cn/item/5f4ba7d1160a154a677ebea3.jpg)](https://pic.imgdb.cn/item/5f4ba7d1160a154a677ebea3.jpg)

[![img](https://pic.imgdb.cn/item/5f4ba7e2160a154a677ec42e.jpg)](https://pic.imgdb.cn/item/5f4ba7e2160a154a677ec42e.jpg)

 和REPEAT循环区别在于WHILE要满足条件才执行循环，REPEAT是满足条件才退出。

### 10.2.10 事件调度器

 5.1版本之后新增的功能，按自定义的时间周期触发某种操作，可以理解为时间调度器。

 最简单的一个事件调度器

 在其实时间的基础上，每隔一个小时触发一次。

[![img](https://pic.imgdb.cn/item/5f4ba8ff160a154a677f1f3f.jpg)](https://pic.imgdb.cn/item/5f4ba8ff160a154a677f1f3f.jpg)

 通过下面命令打开调度器

```
SET GLOBAL event_ scheduler = 1;
```

禁用或者删除调度器

[![img](https://pic.imgdb.cn/item/5f4ba99b160a154a677f489a.jpg)](https://pic.imgdb.cn/item/5f4ba99b160a154a677f489a.jpg)

## 10.3 触发器

### 10.3.1 创建触发器

[![img](https://pic.imgdb.cn/item/5f4ba9f2160a154a677f61f5.jpg)](https://pic.imgdb.cn/item/5f4ba9f2160a154a677f61f5.jpg)

 只能创建在永久表上，不能对临时表创建触发器。

 其中trigger_time是触发器的触发时间，可以使BEFORE或者AFTER，BEFORE指在检查约束前触发。

 trigger_event就是触发器的触发条件，可以使INSERT, UPDATE或DELETE。

 同一个表相同触发时间的相同触发时间，只能定义一个触发器。使用别名OLD和NEW来引用触发器中发生变化的内容。目前还支持行级触发，不支持语句级触发。

### 10.3.2 删除触发器

[![img](https://pic.imgdb.cn/item/5f4baad4160a154a677fa31c.jpg)](https://pic.imgdb.cn/item/5f4baad4160a154a677fa31c.jpg)

### 10.3.3 查看触发器

 用show triggeres命令查看触发器状态，语法等信息

 也可以查看information_schema.triggers

### 10.3.4 触发器的使用

 触发程序不能调用将数据返回客户端的存储程序，也不能使用采用CALL语句的动态SQL语句，但是允许存储程序通过参数将数据返回触发程序。也就是存储过程或函数通过OUT或INPUT类型的参数将数据触发器是可以的。

 不能再触发器中显示或隐式方式开始或结束事务的语句。比如 START TRANS-ACTION,

COMMIT, ROLLBACK

[![img](https://pic.imgdb.cn/item/5f4bab9e160a154a677fe8a4.jpg)](https://pic.imgdb.cn/item/5f4bab9e160a154a677fe8a4.jpg)

# 第11章 事务控制和锁定语句

## 11.1 LOCK TABLES和UNLOCKE TABLES

 LOCK TABLES可以锁定用于当前线程的表。如果表被其他线程锁定，则当前线程会等待，直到可以获取所有锁定为止。

 UNLOCK TABLES 可以释放当前线程获得的任何锁定。当前线程执行另一个LOCK TABLES时，或当服务器的连接被关闭时，所有由当前线程锁定的表被隐含地解锁。



```
LOCK TABLES tbl_ name 
[AS alias] {READ [LOCAL] | [LOW_ PRIORITY] WRITE}
[, tbl_ name [AS alias] {READ [LOCAL] | [LOW_ PRIORITY] WRITE}] . . 
UNLOCK TABLES
```

[![img](https://pic.imgdb.cn/item/5f4cf151160a154a67fa62f3.jpg)](https://pic.imgdb.cn/item/5f4cf151160a154a67fa62f3.jpg)

注意: LOCK TABLES/UNLOCK TABLES 与 LOCK TABLE/UNLOCK TABLE 同义

## 11.2 事务控制

 MySQL通过SET AUTOCOMMIT、START TRANSACTION、COMMIT、ROLLBACK等语句支持本地事务。

```
START TRANSACTION 
| BEGIN [WORK] COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE] 
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE] 
SET AUTOCOMMIT = {0 | 1}
```

默认情况下，MySQL是自动提交的。

START TRANSACTION 或 BEGIN 语句 可以 开始 一项 新的 事务。

COMMIT 和 ROLLBACK 用来 提交 或者 回 滚 事务。

CHAIN 和 RELEASE 子句 分别 用来 定义 在 事务 提交 或者 回 滚 之后 的 操作， CHAIN 会 立即 启动 一个 新事物， 并且 和 刚才 的 事务 具有 相同 的 隔离 级别，

RELEASE 则 会 断开 和 客户 端 的 连接。

SET AUTOCOMMIT 可以 修改 当前 连接 的 提交 方式， 如果 设置 了 SET AUTOC- OMMIT= 0， 则 设置 之后 的 所有 事务 都 需要 通过 明确 的 命令 进行 提交 或者 回 滚。

[![img](https://pic.imgdb.cn/item/5f4cf38b160a154a67fc6ab9.jpg)](https://pic.imgdb.cn/item/5f4cf38b160a154a67fc6ab9.jpg)

[![img](https://pic.imgdb.cn/item/5f4cf39d160a154a67fc759f.jpg)](https://pic.imgdb.cn/item/5f4cf39d160a154a67fc759f.jpg)

 在锁表期间，开始一个事务会造成一个隐含的UNLOCK TABLE在里面

[![img](https://pic.imgdb.cn/item/5f4cf5bb160a154a67fe690c.jpg)](https://pic.imgdb.cn/item/5f4cf5bb160a154a67fe690c.jpg)

 在同一个事务中，最好使用相同存储引擎的表，否则ROLLBACK时需要对非事务类型的表进行特别的处理，因为COMMIT, ROLLBACK只能对事务类型的表进行提交或者回滚。

 通常情况，只对提交的事务记录到二进制的日志中，但是如果一个事务中包含非事务类型的表，那么回滚操作也会被记录到二进制日志中，以确保非事务类型的更新可以被复制到从数据库中。

 所有DDL语句是不能回滚的，部分DDL语句会造成隐式的提交。

 事务中可以通过定义SAVEPOINT，指定回滚事务的一个部分，但不能指定提交事务的一个部分。

 可以定义多个SAVEPOINT,满足不同条件时，回滚不同的SAVEPOINT。需要注意的是，定义了相同名字的SAVEPOINT，会覆盖。可以通过RELEASE SAVEPOINT进行删除SAVEPOINT。

[![img](https://pic.imgdb.cn/item/5f4cf728160a154a67ffca79.jpg)](https://pic.imgdb.cn/item/5f4cf728160a154a67ffca79.jpg)

[![img](https://pic.imgdb.cn/item/5f4cf733160a154a67ffd50b.jpg)](https://pic.imgdb.cn/item/5f4cf733160a154a67ffd50b.jpg)

## 11.3 分布式事务的使用

 5.0.3版本起开始支持分布式事务，InnoDB才能支持。

### 11.3.1 分布式事务的原理

 使用一个或多个资源管理器和一个事务管理器。

- 资源管理器(RM)用于提供通向实物资源的途径，数据库服务器时一种资源管理器，该管理器必须可以提交或回滚RM管理的事务。
- 事务管理器(TM)用于协调作为一个分布式事务一部分的事务。TM与管理的每个事物的RMs进行通信。在一个分布事务中，各个单个事务均是分布式事务的分支事务。

 MySQL执行XA MySQL时，MySQL服务器相当于一个用于管理分布式事务中的XA事务的资源管理器。

 要执行一个分布式事务，必须知道这个分布式事务设计哪些资源管理器，并把每个资源管理器的事务执行到事务可以被提交或回滚时。根据每个资源管理器报告的有关执行情况的内容，这些分支事务必须作为一个原子操作全部提交或回滚。必须要考虑任何组件或者连接网络可能会出现的异常。

 执行分布式事务使用两阶段提交，发生时间在由分布式事务的各个分支需要进行的行动已经被执行之后。

- 在第一阶段中，所有分支都预备好，即他们被TM告知要准备提交。通常，这意味着用于管理分支的各个RM会记录对于被稳定保存的分支的行动。分支指示是否它们可以这么做，这些结果被用于第二阶段。
- 在第二阶段中，TM告知RMs是否要提交或回滚。如果在预备分支时，所有的分支指示它们能够提交，则所有被告知要提交。在预备时，有任何分支指示它们将不能提交，则所有分支被告知回滚。

### 11.3.2 分布式事务的语法

```
XA {START| BEGIN} xid [JOIN| RESUME]
```

 XA START xid用于启动一个带给定xid值的XA事务。每个事务必须有一个唯一的xid，因此该值不能被其他的XA事务使用。

 xid是一个XA事务标识符，用来唯一表示一个分布式事务。由客户端提供或由MySQL服务器生成。

 xid: gtrid [, bqual [, formatID]]

 gtrid是一个分布式事务标识符，相同分布式事务应该使用相同的gtrid，这样可以明确知道XA事务属于哪个分布式事务。

 bqual是一个分支限定符，默认值是空串 。对于一个分布式事务中的每个分支事务，bqual值必须是唯一的。

 formatID是一个数字，用于表示由gtride和bqual值使用的格式，默认值是1。

 下面其他XA语法用到的xid必须和START操作使用的xid相同。

 XA END xid [SUSPEND [FOR MIGRATE]]

 XA PREPARE xid

 使 事务 进入 PREPARE 状态， 也就是 两 阶段 提交 的 第一个 提交 阶段。

 XA COMMIT xid [ONE PHASE]

 XA ROLLBACK xid

 这 两个 命令 用来 提交 或者 回 滚 具体 的 分支 事务。

 也就是 两 阶段 提交 的 第二个 提交 阶段， 分支 事务 被 实际 地 提交 或者 回 滚。

 XA RECOVER

 XA RECOVER 返回 当前 数据库 中 处于 PREPARE 状态 的 分支 事务 的 详细信息。

[![img](https://pic.imgdb.cn/item/5f4e5a36160a154a67a7f01a.jpg)](https://pic.imgdb.cn/item/5f4e5a36160a154a67a7f01a.jpg)

### 11.3.3 存在的问题

 在5.5之前的版本，如果分支事务到达prepare状态，数据库异常重启，重启后，可以选择对分支事务进行提交或回滚操作，但是即使选择提交事务，该事务也不会被写入BINLOG。可能导致使用BINLOG恢复时丢失部分数据。如果存在复制的从库，则有可能导致主从数据库的数据不一致。一下演示了这个过程。

[![img](https://pic.imgdb.cn/item/5f4e5b8f160a154a67a84940.jpg)](https://pic.imgdb.cn/item/5f4e5b8f160a154a67a84940.jpg)

[![img](https://pic.imgdb.cn/item/5f4e5baa160a154a67a85056.jpg)](https://pic.imgdb.cn/item/5f4e5baa160a154a67a85056.jpg)、

 在MySQL5.7中，已经解决了XA事务严格持久化的问题，在session断开和实例崩溃的情况下，事务都不会自动回滚，在XA PREPARE时，之前的事务信息就会被写入BINLOG并同步到备库中。

 5.7中，在XA事务在结束之后，提交之后，不允许进行查询。

 总之，MySQL的分布式事务还存在一些问题，在数据库或应用异常的情况下，可能会导致分布式事务的不完整或需要人工介入处理。

# 第12章 SQL中的安全问题

## 12.1 SQL注入简介

 SQL注入就是利用某些数据库外部接口将用户信息插入到实际的数据库操作语言当中，从而达到入侵数据库乃至操作系统的目的，甚至可以获得数据库管理员权限。而且SQL注入也很难防范。

 SQL注入例子

（1） 创建 用户 表 user：

```
CREATE TABLE user ( 
userid int( 11) NOT NULL auto_ increment, 
username varchar( 20) NOT NULL default '', 
password varchar( 20) NOT NULL default '', 
PRIMARY KEY (userid) ) 
TYPE= MyISAM AUTO_ INCREMENT= 3 ;
```

（2） 给用户 表 user 添加 一条 用户 记录：

```
INSERT INTO ` user` VALUES (1, 'angel', 'mypass');
```

（3） 验证 用户 root 登录 localhost 服务器：

```
<? php $ servername = "localhost"; 
	$ dbusername = "root"; 
	$ dbpassword = "";
	$ dbname = "injection"; 
	mysql_ connect($ servername,$ dbusername,$ dbpassword) or die ("数据库 连接 失败"); 
	$ sql = "SELECT * FROM user WHERE username='$ username' AND password= '$password'";
	$ result = mysql_ db_ query($ dbname, $ sql); 
	$ userinfo = mysql_ fetch_ array($ result); 
	if (empty($ userinfo)) { echo "登录 失败"; } else { echo "登录 成功"; } echo "<p> SQL Query:$ sql< p>"; ?>
```

（4） 然后 提交 如下 URL：

```
http:// 127. 0. 0. 1/ injection/ user. php? username= angel' or '1= 1
```

 发现这个URL可以登录系统。同样也可以利用SQL的注释语句实现SQL注入。

```
http:// 127. 0. 0. 1/ injection/ user. php? username= angel'/*
http:// 127. 0. 0. 1/ injection/ user. php? username= angel'#
```

 将后面的局语句都注释掉，只根据用户名没有根据密码的URL都成功进行了登录。利用or和朱师傅不同在于，前者是利用逻辑运算，后者是MySQL的特性，比逻辑运算简单得多。

## 12.2 应用开发中可以采取的措施

### 12.2.1 PrepareStatemen+Bind-Variables

 MySQL不存在共享池的概念，所以再MySQL上绑定变量的最大好处主要就是避免SQL注入。

### 12.2.2 使用应用程序提供的转换函数

 很多应用程序的接口都提供了对特殊字符的转换函数。恰当地使用这些函数，可以防止应用程序用户输入生成非期望的语句。

### 12.2.3 自定义函数进行校验。

- 整理 数据 使之 变得 有效；
- 拒绝 已知 的 非法 输入；
- 只 接受 已知 的 合法 输入。

下面的正则表达式可以提供一个验证函数：

已知 非法 符号 有：“’ ”、“;”、“=”、“(”、“)”、“/*”、“*/”、“%”、“+”、“”、“>”、“<”、“–”、“[” 和“]”。

由此， 可以 构造 如下 正 则 表达式：

(|'|(%27)|;|(% 3b)|=|(% 3d)|(|(% 28)|)|(% 29)|(/*)|(% 2f% 2a)|(\*/)|(% 2a% 2f)|+| (%2b)|<|(% 3c)|>|(% 3e)|(–))|[|% 5b|]|% 5d)

# 第13章 SQL Mode及相关问题

## 13.1 MySQL SQL Mode简介

- 通过设置SQL Mode，可以完成不同严格程度的数据校验，有效地保护数据准确性。
- 通过设置SQL Mode为ANSI模式，来保证大多数SQL符合标准的SQL语法，这样应用在不同数据库之间进行迁移时，则不需要对业务SQL进行较大修改
- 在不同数据库之间进行数据迁移之前，通过设置SQL Mode可以使MySQL上的数据更方便地迁移到目标数据库中。

 在MySQL5.7中，SQL Mode有了较大的变化，查询默认SQLMode为ONLY_FULL_GROUP_BY, STRICT_TRANS_TABLES, NO_ZEOR_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_AUTO_CREATE_USER和NO_ENGINE_SUBSTITUTION（不同小版本可能略有区别)。

| sql_mode值                     | 描述                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| **ONLY_FULL_GROUP_BY**         | group by中没出现的列，在select， having， order by中会被拒绝 |
| **STRICT_TRANS_TABLES**        | 非法日期，超过字段长度的值插入时，会报错                     |
| **NO_ZEOR_IN_DATE**            | 日期中针对月份和日期部分，如果为0，有不同执行逻辑 1.disable：可以正常插入 2.enable:可以正常插入，有警告，如果mode中包含STRICT_TRANS_TABLES，但是可以通过ignore关键字写入’000-00-00’ |
| **NO_ZERO_DATE**               | 针对日期’0000-00-00’ 1.disable：可以正常插入 2.enable:可以正常插入，有警告，如果mode中包含STRICT_TRANS_TABLES，但是可以通过ignore关键字写入’000-00-00’，有警告 |
| **ERROR_FOR_DIVISION_BY_ZERO** | 除数为0，包括模运算 1.disable:插入NULL，没有警告 2.enable:插入NULL，有警告，如果mode中包含STRICT_TRANS_TABLES，则拒绝写入，可以通过ignore子句写入NULL，有警告 |
| **NO_AUTO_CREATE_USER**        | 防止使用不带密码的子句来创建一个用户                         |
| **NO_ENGINE_SUBSTITUTION**     | 执行create table或altertable，如果指定了不支持的(包括disable或者未编译)的存储引擎，是否自动替换为默认引擎 1.disable:craete table自动替换后执行，alter table不会执行，都有警告 2.enable：两个命令直接报错 |

 5.7.5版本之后，最大区别是在SQL Mode设置中，加了严格的事物表模式。

(1) 查看SQLMode命令

```
select @@sql_mode
```

## 13.2 SQL Mode常见功能

(1) 校验日期合法性。

[![img](https://pic.imgdb.cn/item/5f4f9b0a160a154a67f57c8d.jpg)](https://pic.imgdb.cn/item/5f4f9b0a160a154a67f57c8d.jpg)

 ANSI模式下可以插入非法日期，但是插入值变为’0000-00-00’,TRADITIONAL模式不可以。

（2）启用NO_BACKSLASH_ESCAPES模式，使\变成普通字符。

（3）启用PIPES_AS_CONCAT模式，将’||’视为字符串连接符号。

## 13.3 常用SQL MODE

[![img](https://pic.imgdb.cn/item/5f4f9d21160a154a67f6251b.jpg)](https://pic.imgdb.cn/item/5f4f9d21160a154a67f6251b.jpg)

## 13.4 SQL Mode在迁移中如何使用

[![img](https://pic.imgdb.cn/item/5f4f9d50160a154a67f63886.jpg)](https://pic.imgdb.cn/item/5f4f9d50160a154a67f63886.jpg)

# 第14章 MySQL分区

## 14.1 分区概述

 有利于管理非常大的表，采用分而治之的逻辑。引入了分区键的概念。分区间根据某个区间值（或范围值）、特定值列表或者HASH函数执行数据的聚集，让数据规则地分布在不同的分区中，让一个大对象变成一些小对象。

 优点：

- 和单个磁盘或文件系统分区相比，可以存储更多数据
- 优化数据查询。在Where语句中包含分区条件时，可以只扫描必要的一个或多个分区来提高效率；在涉及SUM()和COUNT()这类聚合函数时，可以容易地在每个分区并行处理，注最终只需要汇总所有分区得到的数据。在5.7版本中，可以通过类似于SELECT * FROM T PARTITION(p0, p1)来显示地指定查询分区。
- 对于已经过期或不需要存储的数据，可以通过删除与这些数据有关的分区来快速删除数据
- 跨多个磁盘来分散查询数据，提高吞吐量。

5.7中，通过二进制包安装会默认包含分区支持，通过源代码编译安装，需要在编译时指定参数DWITH_PARTITION_STORAGE_ENGINE。

 通过SHOW PLUGINS命令或查询PLUGINS字典表来确认当前MYSQL是否支持分区。

 支持大部分存储引擎(比如MyISAM、InnoDB、Memory等)创建分区表，不支持MERGE, CSV和FEDERRATED这三种引擎。

 5.7版本中，同一个分区表的分区必须使用相同的存储引擎，分区数量不超过8192.

 分区表设置存储引擎，只能使用[STORAGE]ENGINE子句，必须列在CREATE TABLE其他任何分区选项之前

```
CREATE TABLE emp 
(empid INT, 
salary DECIMAL( 7, 2), 
birth_ date DATE) 
->ENGINE= INNODB 
-> PARTITION BY HASH( MONTH( birth_ date) )
-> PARTITIONS 6;
```

注意：MySQL分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区。反过来也一样。MySQL分区表上创建的索引一定是本地索引。

## 14.2 分区类型

 5.7主要有以下6种：

- RANGE分区：基于一个连续区间范围，把数据分配到不同的分区
- LIST分区：基于枚举出的值列表分区，RANGE是基于给定的连续区间分区
- COLUMNS：分区键可以是多了，也可以是非整数。
- HASH: 基于给定的分区个数，把数据取模分配到不同的分区。
- KEY分区：类似于HASH分区，但使用MySQL提供的hash函数。
- 子分区：也叫符复合分区或组合分区，即在主分区下再做一层分区，将数据再次分割。

 5.7中RANGE,LIST,HASH分区的分区键必须是INT类型，或者通过表达式返回INT类型，KEY和COLUMNS出来，可以使用其他类型的列（BLOB和TEXT除外）作为分区键。

 如果希望在RANGE或LIST类型分区上使用非INT键做分区键，可以选择COLUMNS分区。无论哪种MySQL分区类型，要么分区表上没有唯一键/主键，要么主键/唯一键包含分区间。

 分区名字基本上遵循MySQL标识符的原则。在MySQL中，对于库名和表明是否大小写铭感，可以通过下面的方法进行查看。

```
show variables like '%lower_case%'
```

 查询出的lower_case_file_system是一个不可修改的变量，代表操作系统是否大小写敏感，OFF代表敏感，ON代表不敏感。如果操作系统大小写敏感，那么库名和表名是否大小写敏感就由lower_case_table_names变量决定，1不敏感。

[![img](https://pic.imgdb.cn/item/5f50e568160a154a673d02c2.jpg)](https://pic.imgdb.cn/item/5f50e568160a154a673d02c2.jpg)

 注意，列名，别名，分区名这些是不区分大小写的。

### 14.2.1 RANGE分区

```
CREATE TABLE emp ( 
->　 id INT NOT NULL, 
->　 ename VARCHAR( 30), 
->　 hired DATE NOT NULL DEFAULT '1970- 01- 01', 
->　 separated DATE NOT NULL DEFAULT '9999- 12- 31', 
->　 job VARCHAR( 30) NOT NULL, 
->　 store_ id INT NOT NULL 
-> )
-> PARTITION BY RANGE (store_ id) ( 
->　 PARTITION p0 VALUES LESS THAN (10), 
->　 PARTITION p1 VALUES LESS THAN (20), 
->　 PARTITION p2 VALUES LESS THAN (30) 
-> );
```

 此时，插入store_id大于30的行，就会出现错误。可以在分区时使用VALUES LESS THAN MAXVALUE，提供给所有大于明确最高值的值，MAXVALUE表示最大的可能整数值。

 支持在VALUES LESS THAN子句中使用表达式。

```
CREATE TABLE emp_ date ( 
->　 id INT NOT NULL, 
->　 ename VARCHAR( 30), 
->　 hired DATE NOT NULL DEFAULT '1970- 01- 01', 
->　 separated DATE NOT NULL DEFAULT '9999- 12- 31', 
->　 job VARCHAR( 30) NOT NULL, 
->　 store_ id INT NOT NULL 
-> )
-> PARTITION BY RANGE (YEAR( separated)) ( 
->　 PARTITION p0 VALUES LESS THAN (1995), 
->　 PARTITION p1 VALUES LESS THAN (2000), 
->　 PARTITION p2 VALUES LESS THAN (2005) -> );
```

 从5.5版本开始，提供了RANGE COLUMNS支持非整数分区。

```
CREATE TABLE emp_ date( 
->　 id INT NOT NULL, 
->　 ename VARCHAR( 30), 
->　 hired DATE NOT NULL DEFAULT '1970- 01- 01', 
->　 separated DATE NOT NULL DEFAULT '9999- 12- 31', 
->　 job VARCHAR( 30) NOT NULL, 
->　 store_ id INT NOT NULL -> ) 
-> PARTITION BY RANGE COLUMNS (separated) (
->　 PARTITION p0 VALUES LESS THAN ('1996- 01- 01'),
->　 PARTITION p1 VALUES LESS THAN ('2001- 01- 01'), 
->　 PARTITION p2 VALUES LESS THAN ('2006- 01- 01') -> );
```

 RANGE分区特别适用于以下两种情况：

- 当删除过期数据时，只需要简单的ALTER TABLE DROP PARTITION p0来删除p0分区中的数据。
- 经常运行包含分区键的查询，MySQL可以很快确定只有某一个或者某些分区需要扫描，因为其他分区不可能包含符合该WHERE子句的任何记录。

### 14.2.2 LIST分区

```
mysql> CREATE TABLE expenses ( 
->　 expense_ date DATE NOT NULL, 
->　 category INT, 
->　 amount DECIMAL (10, 3) 
-> )PARTITION BY LIST( category) ( 
->　 PARTITION p0 VALUES IN (3, 5), 
->　 PARTITION p1 VALUES IN (1, 10), 
->　 PARTITION p2 VALUES IN (4, 9), 
->　 PARTITION p3 VALUES IN (2), 
->　 PARTITION p4 VALUES IN (6) -> );
```

 支持非整数值，使用LIST COLUMNS。

### 14.2.3 COLUMNS分区

 5.5引入的。

 支持整数、字符串、日期时间三大类型。

注意：5.7中，分区仅支持一个或多个字段名作为分区间，不支持表达式作为分区键。

 COLUMNS还支持多列分区。

[![img](https://pic.imgdb.cn/item/5f50e97a160a154a673e6bff.jpg)](https://pic.imgdb.cn/item/5f50e97a160a154a673e6bff.jpg)

### 14.2.4 HASH分区

 支持两种HASH分区，常规HASH分区和线性HASH分区(LINEAR HASH)分区。常规HASH使用取模，线性HASH使用的是一个线性的2的幂的运算法则。

```
CREATE TABLE emp ( 
->　 id INT NOT NULL, 
->　 ename VARCHAR( 30),
->　 hired DATE NOT NULL DEFAULT '1970- 01- 01', 
->　 separated DATE NOT NULL DEFAULT '9999- 12- 31', 
->　 job VARCHAR( 30) NOT NULL, 
->　 store_ id INT NOT NULL-> ) 
-> PARTITION BY HASH (store_ id) PARTITIONS 4;
```

 表达式expr可以使MySQL中有效的任何函数或其他表达式，只要它们返回一个非常数也非随机的整数。每当插入/更新/删除/一行数据，这个表达式都需要计算一次，这意味着非常复杂的表达式可能会引发性能问题，也不推荐涉及多列的哈希分区。

 常规HASH看上去不错，但是调整分区个数时的代价很大。为了减少这个待见，提供了线性HASH。

 唯一区别就是HASH前添加关键字 LINEAR。

 优点是，在维护分区时，MySQL能够处理得更加迅速，缺点是分布不太均匀。

### 14.2.5 KEY分区

 非常类似于HASH分区，不过HASH分区允许使用用户自定义的表达式，而KEY分区不可以，而KEY分区支持除BLOB和Text外其他类型的列作为分区键。

 使用PARTITION BY KEY(expr)来创建一个KEY分区，expr可以使0个或多个字段名的列表。不指定分区键，默认说先选择主键进行分区，如果没有主键，会选择非空唯一键作为分区键。没有主键和唯一键情况下，不能指定分区键。

 和HASH一样，可以使用LINEAW关键字。

### 14.2.6 子分区

 对分区表每个分区再次分割，又被人称为复合分区。

 5.7支持对已经通过RANGE或LIST分区了的表再进行分区。子分区支持HASH或KEY分区。

```
CREATE TABLE ts (
id INT, purchased DATE) 
->　 PARTITION BY RANGE( YEAR( purchased)) 
->　 SUBPARTITION BY HASH( TO_ DAYS( purchased)) 
->　 SUBPARTITIONS 2 ->　( 
->　 　 PARTITION p0 VALUES LESS THAN (1990), 
->　 　 PARTITION p1 VALUES LESS THAN (2000), 
->　 　 PARTITION p2 VALUES LESS THAN MAXVALUE 
->　);
```

 复合分区适用于保存非常大量的数据记录：

- 每个分区必须具有相同数量的子分区；
- 如果要显示指定子分区，则每个分区都要显示指定；
- 子分区的名称在表中是唯一的

[![img](https://pic.imgdb.cn/item/5f50f18e160a154a674091a7.jpg)](https://pic.imgdb.cn/item/5f50f18e160a154a674091a7.jpg)

### 14.2.7 MySQL分区处理NULL值的方式

 MySQL不禁止分区键值上使用NULL值，一般情况下把NULL当成0值或者最小值进行处理。

- RANGE分区中，NULL被当做最小值来处理；LIST分区，NULL值必须出现在枚举列表中；HASH或KEY分区中，NULL值会被当成零值来处理。

## 14.3 分区管理

### 14.3.1 RANGE和LIST分区管理

 在添加、删除和重新定义分区的处理上，RANGE分区和LIST分区非常相似。

 删除可以用ALTER TABLE DROP PARTITION语句来实现，比如

```
ALTER TABLE emp_date drop partition p2
```

 增加分区可以用ALTER TABLE ADD PARTITION语句来实现。对于RANGE分区来说，只能通过ADD PARTITION方式添加新分区到分区列表的最大一端。

```
ALTER TABLE emp_date add partition(partition p5 values less than (2025));

alter table expenses add partition(partition p6 values in (6, 11));
```

也提供了在不丢失数据的情况，通过重新定义分区的语句ALTER TABLE REORGANIZE PARTITION INTO重新定义分区。

```
alter table emp_date reorganize partition p3 into(
	partition p2 values less than (2005),
	partition p3 values less than (2015)
)
```

也可以合并分区

```
alter table emp_date reorganize partition p3,p4 into(
	partition p2 values less than (2005)
)
```

注意：类似于重定义RANGE分区，重定义LIST分区时，只能够重新定义相邻的分区，不能跳过LSIT分区进行重新定义，通识重新定义的分区区间必须和原分区区间覆盖相同的区间。也不能修改分区类型。

### 14.3.2 HASH和KEY分区管理

 不能以之前的方式进行删除，而可以通过ALTER TABLE COALESCE PARTITION的语句来合并HASH分区或KEY分区。

```
ALTER TBALE emp COALESCE partition 2
```

 COALESCE 不能用来增加数量。

 要增加分区用

```
ALTER TBALE emp add partition 8;//最终变成2+8 = 10个
```

### 14.3.3 交换分区

 5.6版本添加了交换分区功能，使用ALTER TABLE pt EXCHANGE PARTITION p WITH TABLE nt命令，可以实现将分区表pt中的一个分区或者子分区p中的数据与普通表nt中的数据进行交换。

- 表nt不能是分区表，由于交换分区不能通过分区对分区的方式进行，如果有这种需求，可以用一个普通表作为中间表，通过交换两次分区来实现；

- 表nt不能是临时表

- 表pt和nt的结构，除了分区要一致，包括索引。

- nt上不能有外键，也不能有其他表的外键依赖nt

- nt表的所有数据，都应该在分区定义的范围内。

  5.7版本如果确定都在范围内，可以增加WITHOUT VALIDATION来跳过。

# 第15章 SQL优化

## 15.1 优化SQL语句的一般步骤

### 15.1.1 通过show status命令了解各种SQL的执行效率

 通过show [session|global]status 可以提供服务器状态信息，默认使用session级。

[![img](https://pic.imgdb.cn/item/5f56247c160a154a6760d5a1.jpg)](https://pic.imgdb.cn/item/5f56247c160a154a6760d5a1.jpg)

 Com_xxx表示每个xxx语句执行的次数，通常比较关心:

 Com_select, Com_insert, Com_update, Com_delete.

 下面的只针对InnoDB：

- Innodb_rows_read: select查询返回的行数
- Innodb_rows_insert：插入的行数
- Innodb_rows_update：更新的行数
- Innodb_rows_delete：删除的行数

 对于事务性的应用，Com_commit和Com_rollback可以了解事务提交和回滚的情况。

 此外，以下参数便于用户了解数据库的情况

- Connection：实录连接MySQL服务器的次数
- UpTime：服务器工作时间
- Slow_queries:慢查询次数

### 15.1.2 定位执行效率较低的SQL语句

- 将slow-query-log设置为1，会将所有执行时间超过long_query_time参数所设定的阈值SQL，写入slow_query_log_file参数指定的文件中。
- 慢查询日志在查询结束后才会结束，可以使用show processlist命令查看当前MySQL在进行的线程。

### 15.1.3 通过EXPLAIN分析低效SQL的执行计划

```
explain select sum(amount) from customer a, payment b where 1=1 and a.customer_id = b.customer_id and email = 'JANE.BENNETT@sakilacustomer.org'
*************************** 1. row *************************** 
id: 1 
select_type: SIMPLE 
table: a
type: ALL 
possible_keys: PRIMARY 
key: NULL 
key_len: NULL 
ref: NULL 
rows: 583 
Extra: Using where 
*************************** 2. row *************************** 
id: 1 
select_ type: SIMPLE 
table: b 
type: ref 
possible_keys: idx_fk_customer_id 
key: idx_fk_customer_id 
key_len: 2 
ref: sakila.a.customer_id 
rows: 12 
Extra: 
2 rows in set (0. 00 sec)
```

- select_type: 表示SELECT的类型，常见取值有SIMPLE表(简单表，即不使用表连接或子查询)、PRIMARY(主查询，即外层的查询)、UNION(UNION中第二个或后面的查询语句)、SUBQUERY(子查询的第一个SELECT)等。
- table:输出结果的表
- type：表示MySQL在表中找到所需行的方式，或叫访问类型。

[![img](https://pic.imgdb.cn/item/5f562917160a154a67622404.jpg)](https://pic.imgdb.cn/item/5f562917160a154a67622404.jpg)

 以上是常见类型，性能最差到最好

1. type=ALL, 全表扫描；
2. index:索引全扫描；
3. range:索引范围扫描，常见于<、<=、>、>=、between等操作符；
4. ref: 使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的行；
5. eq_ref：类似于ref，不过使用的是唯一索引；
6. const/system：单标中最多的一个匹配行，查询非常迅速，其他列的值可以被优化器当前查询中当成常量来处理，例如，跟住助教或唯一索引进行查询。
7. NULL：不用访问表或索引，直接就能够得到结果。

 除了以上，还有其他值:ref_or_null(与ref类似，包含NULL的查询)、index_merge(索引合并优化)、unique_subquery(in后面是一个查询主键字段的子查询), index_subquery

- possible_keys: 表示查询时可能用到的索引。
- key：表示实际使用的索引
- key_len：使用索引字段的长度
- rows:扫描行的数量
- Extra：执行情况的说明。

 5.1版本开始支持分区，通识explain也增加了对分区的支持。可以通过explain partions命令查看SQL所访问的分区。

### 15.1.4 通过show profile分析SQL

 从5.0.37版本增加了对show profiles和show profile语句支持。通过have_profiling参数，查看当前MySQL是否支持。

```
select @have_profiling
```

 默认关闭，可以通过set语句在Session级别开启profiling

```
set profilling=1
```

 通过profile，用户能更清楚地了解SQL的执行过程。

```
show profiles
```

[![img](https://pic.imgdb.cn/item/5f56317b160a154a676427d9.jpg)](https://pic.imgdb.cn/item/5f56317b160a154a676427d9.jpg)

通过show profile for query 语句可以查看执行过程中线程的每个状态和消耗时间。

```
show profile for query 4;
```

[![img](https://pic.imgdb.cn/item/5f5632e0160a154a676494ec.jpg)](https://pic.imgdb.cn/item/5f5632e0160a154a676494ec.jpg)

为了更清晰地看到排序结果，可以查询information_schema.profiling表，并按照时间做个DESC排序。

 在获取最消耗时间的线程状态后，MySQL进一步支持all、cpu、block io、context switch、page faults等明细类型查看MySQL在使用什么资源上耗费了过高的时间。

```
show profile cpu for query 4
```

 如果对源代码感兴趣，还可以使用show profile source for query查看SQL解析执行过程中每个步骤对应的源码的文件、函数名以及具体的源文件行数。

 MySQL5.6之后则通过trace文件进一步向我们展示了优化器是如何选择执行计划的。

 注意：5.7版本中，profile已经不建议使用，而使用performance schema中一系列性能视图来代替，详细参考第20章。

### 15.1.5 通过trace分析优化器如何选择执行计划

 5.6版本开始提供了对SQL的跟踪trace。

 使用方式：首先打开trace，设置格式为JSON，设置trace最大能够使用的内存大小，避免解析过程中因默认内存过小而不能完整显示。

```
SET OPTIMIZER="enabled=on", END_MARKERS_IN_JSON=on;
SET OPTIMIZER_TRACE_MAX_MEM_SIZE=1000000;
```

接下来执行想做trace的SQL语句

然后检查INFORMATION_SCHEMA.OPTIMIZER_TRACE就可以知道MySQL是如何执行SQL语句的：

```
SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE
```

会输出一个跟踪文件。

### 15.1.6 确定问题并采取相应措施

 比如如果全表扫描可以考虑加索引。

## 15.2 索引问题

### 15.2.1 索引的存储分类

 索引在MySQL的存储引擎层中实现的。

- B-Tree索引：最常见的索引类型，大部分引擎都支持B树索引；
- HASH索引：只有Memory/NDB支持，使用场景简单；
- R-Tree索引(空间索引)：空间索引时MyISAM的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少，不做特别介绍。
- Full-text(全文索引)：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从MySQL5.6版本开始提供对全文索引的支持。

 8.0.11还不支持函数索引，但是可以通过两种实现函数索引的功能。

（1）前缀索引，即对列的前面某一部分进行索引，可以大大缩小索引文件的大小，但是也有缺点，orderBy和groupBy操作的时候无法使用。

```
create index idx_title on film(title(10));
```

(2) 虚拟列索引。5.7版本后支持，可以通过创建虚拟列索引来实现函数索引的功能。

```
alter table salaries add column salary_by_1k int generated always as (round(salary/1000));
alter table salaries add key idx_salary_by_1k(salary_by_1k);
```

[![img](https://pic.imgdb.cn/item/5f564492160a154a67699ece.jpg)](https://pic.imgdb.cn/item/5f564492160a154a67699ece.jpg)

### 15.2.2 MySQL如何使用索引

 B-Tree索引是最常见的索引，构造类似二叉树，能根据键值提供一行或一个行集的快速访问。

[![img](https://pic.imgdb.cn/item/5f577a2d160a154a67ac147b.jpg)](https://pic.imgdb.cn/item/5f577a2d160a154a67ac147b.jpg)

#### 1.MySQL中能够使用索引的经典场景

(1) 匹配全值

(2) 匹配值的范围查找

(3) 匹配最左前缀

(4) 仅仅对索引进行查询

(5) 匹配列前缀

(6) 能够实现索引匹配部分精确而其他部分进行范围匹配

(7) 如果列名是索引

(8) 5.6引入了Index Condition Pushdown(ICP)特性，进一步优化了查询，Pushdown代表操作下方，某些情况下条件过滤下方到存储引擎。[![img](https://pic.imgdb.cn/item/5f577cc3160a154a67ad0ffe.jpg)](https://pic.imgdb.cn/item/5f577cc3160a154a67ad0ffe.jpg)

[![img](https://pic.imgdb.cn/item/5f577cdc160a154a67ad17a2.jpg)](https://pic.imgdb.cn/item/5f577cdc160a154a67ad17a2.jpg)

#### 2.存在索引但不能使用索引的典型场景

(1) 以%开头的LIKE查询不能利用B-Tree索引；

一般采用轻量级的解决方式，一般情况下，索引会比表小，扫描索引要比扫描表更快，而InnoDBInnoDB 表上 二级 索引 idx_ last_ name 实际上 存储 字段 last_ name 还 有主 键 actor_ id， 那么 理想 的 访问 方式 应该 是 首先 扫描 二级 索引 idx_ last_ name 获得 满足 条件 last_ name like ‘%NI%’ 的 主 键 actor_ id 列表， 之后 根据 主 键 回 表 去 检索 记录， 这样 访问 避 开了 全 表 扫描 演员 表 actor 产生 的 大量 IO 请求。

```
explain select * from (select actor_ id from actor where last_ name like '%NI%') a, actor b where a. actor_ id = b. actor_ id
```

(2) 数据类型出现隐式转换也不会使用索引；

(3) 符合索引情况下，加入查询条不包含列最左边部分，不会使用符合索引；

(4) MySQL估计全表扫描更快。

### 15.2.3 查看索引使用情况

 如果索引正在工作, Handler_read_key的值将很高，这个值代表了一个行被索引值读的次数。

```
show status like 'Handler_read%'
```

## 15.3 两个简单实用的优化方法

### 15.3.1 定期分析表和检查表

 分析表的语句。

```
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tal_nam[, tbl_name]...
```

 本语句用于分析和存储表的关键字分布，分析结果可以使得系统得到准确的统计信息，使得SQL能够生成正确的执行计划。分析期间，使用一个读取锁定对表进行锁定。这对MyISAM、BDB、InnoDB表有作用。对MyISAM，本语句与使用myisamchk -a相当。

 检查表语法如下

```
check TABLE tbl_name[, tbl_name] ... [option]... option={QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
```



### 15.3.2 定期优化表

```
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tal_nam[, tbl_name]...
```

 对于InnoDB的表，设置innodb_file_per_table参数，设置为独立表空间模式，每个数据库的每个表都会生成一个ibd文件，用于存储表的数据和索引，一定程度上减轻InnoDB表的空间回收问题。删除大量数据后，InnoDB表可以通过 alter table但是不该表引擎的方式来回收不用的空间。

```
alter table payment engine=innodb
```

 注意：ANALYZE, CHECK, OPTIMIZE, ALTER TABLE 执行期间将对表进行锁定，因此一定注意要在数据库不繁忙时使用。

## 15.4 常用SQL的优化

### 15.4.1 大批量插入数据

 当时用load命令导入数据时，适当设置可以提高导入速度。

 如果MyISAM引擎表

```
ALTER TABLE tbl_name DISABLE KEYS;
loading the data
ALTER TABLE tbl_name ENABLE KEYS;
```

 通过关闭MyISAM表非唯一索引的更新，来提高速度。注意导入空表时无须这样设置。

 对于InnoDB的表

(1) InnoDB类型按主键存顺序保存，所以导入数据按主键的顺序排列，可以有效提高效率。

(2) 导入数据前执行 SET UNIQUE_CHECKS=0, 关闭唯一性校验，导入结束后，SET UNIQUE_CHECKS=1进行恢复

(3)SET AUTOCOMMIT=0关闭，导入结束后打开，SET AUTOCOMMIT=1

### 15.4.2 优化INSERT语句

 考虑以下几种优化方式

- 同时从同一客户插入很多行，尽量使用多个值表的INSERT语句，这种方式大大缩减客户端与数据库的连接、关闭等消耗，效率更高。

  ```
  insert into test values(1,2),(1,3)
  ```

- 不同客户插入多行，可以使用INSERT DELAYED语句得到更高的速度。DELAYED语句是让INSERT语句马上执行，其实数据都被放在内存的队列中，并没有真正写入磁盘，要更快。LOW_PRIORITY相反，在所有其他用户读写完成后，才进行插入。

- 将索引文件和数据文件放在不同的磁盘上

- 如果进行批量插入，可以通过增加bulk_insert_buffer_size变量的值来提高速度，只能对MyISAM使用

- 当一个文本文件装载一个表时，使用LOAD DATA INLINE。通常比使用很多INSERT语句快20倍。

### 15.4.3 优化ORDER BY语句

#### 1.MySQL有两种排序方式

 第一种通过有序所有顺序扫描直接返回有序数据，效率较高

 第二种通过对返回的数据进行排序，也就是常说的Filesort排序，所有不是通过索引直接返回排序结果的排序都叫Filesort排序。

 Filesort是通过相应的排序算法，将取得的数据在sort_buffer_size系统变量设置的内存排序区中进行排序，如果内存装载不下，它就会将磁盘上的数据进行分开，在对每个数据块进行排序，然后将各个块合并成有序的结果集。

 优化目标：尽量减少额外的排序，通过索引直接返回有序数据。

 以下情况不能使用索引：

- order by字段混合ASC和DESC
- 用于查询行的关键字与ORDER BY中使用的不同
- 对不同的关键字使用ORDER BY

#### 2.Filesort的优化

 对于Filesort，MySQL有两种排序算法

- 两次扫描法(TWO Passes): 首先根据条件去除排序字段和行指针信息，之后再排序区sort buffer排序。如果sort buffer不够，则在临时表Temporary Table中存储排序结果。完成后根据行指针回表读读取数据。该算法是4.1之后采用的算法，需要两次访问数据，第一次获取排序字段和行指针信息，第二次根据行指针获取记录，尤其是第二次读取操作可能导致大量随机I/O操作；优点是排序时的内存开销比较小。
- 一次扫描算法(Single Pass):一次性读取满足条件的行的所有字段，然后在排序区sort buffer中排序后直接输出结果表。排序时候的内存开销比较大，但是排序效率比两次扫描算法高。

 MysQL通过比较系统变量max_lenth_for_sort_data大小和Query语句去除字段总大小来判断使用哪种算法，如果max_lenth_for_sort_data更大，使用第二种，否则使用第一种。

 适当加大max_lenth_for_sort_data会提高效率。

### 15.4.4 优化Group By语句

 默认情况下，MySQL对所有 Group by字段排序。

 如果查询Group By但用户想要避免排序结果的小号，则可以指定ORDER BY NULL禁止排序。

### 15.4.5 优化JOIN操作

 MySQL对于多表JOIN目前只支持一种算法—Nested-Loop Join(NLJ)。原理分厂简单，就是内外两层循环，对于外循环的每条记录，都要在内循环中做一次检索。伪代码如下：

```
for each row in t1 matching range {
	for each row in t2 maching reference key {
		if row satisfied join condition, send to client
	}
}
```

 其中t1和t2表进行join，t1通过范围扫描取每条记录作为外循环，t2通过关键字段在表中做扫描，满足条件则返回客户端；不断重复这个过程直到外循环结束。

 NLJ性能高低主要取决于两方面：一是外循环结果集大小，二是内循环扫描数据的效率，常见的优化方案是在驱动表上尽可能where条件创建合适索引，是的外循环的结果集更小，读取效率更高；内循环换是为了提高扫描效率，通常需要在关联字段上加索引。

 有两种情况，NLJ的性能会有比较明显的下降：

- 外循环结果集打，导致访问内循环的io次数非常多
- 内循环的关联字段并不是唯一索引，而是普通的辅助索引。如果访问的数据列不再辅助索引上，此时通常需要再次回表，通过辅助索引的主键找到聚合索引实际的数据，而回表会导致大量随机io产生，导致性能下降明显。

 为了优化这两个为题，先后推出了两个NLJ的变种BNL(Block Nested-Loop Join)和BKA(Batched Key Access)

#### 1.BNL

 在较早版本就引入，算法伪代码如下。

```
for each row in t1 maching range {
	for each row in t2 maching reference key {
		store used columns from t1, t2 in join buffer
		if buffer in null {
			for each row in t3 {
				for each t1, t2 combinationin join buffer {
					if row satisfies join conditions, send to client
				}
			}
		}
	}
}
for each row in t3 {
	for each t1, t2 combinationin join buffer {
		if row satisfies join conditions, send to client
	}
}
```

 通过缓存外层循环读的行，来降低内层表的读取次数。

 5.7中，BNL优化器默认打开。

 BNL性能虽然大幅提高，但是使用条件比较苛刻，只有当join类型是all/index/range时才可以，也就是内标不使用索引或索引效率很低时才不得不使用。buffer的大小由参数join_buffer_size进行设置。

#### 2. MRR & BKA

 大多数情况join操作通常是通过效率较高的索引来做ref或eq_ref方式连接，这种情况下，BNL无法使用。为了优化这种更常见的join，MySQL引入了MRR和BKA。

 MRR(Multi Range Read) 是MySQL5.6引入的特性。优化目的是减少磁盘随机访问。InnoDB由于是狙击索引的特性，如果查询使用辅助索引，并且用到表中非索引列，需要回表读取数据做后续处理。过于所及的回表会伴随大量的随机IO。MRR是通过范围扫描将数据存储read_rnd_buffer_size，然后对齐按照Primary Key(Row ID)进排序，最后使用排序好的数据进行顺序回表。

 MRR特性在单表和多表join中都可以使用。单标通常通过范围查询；多表join方式如果是ref/eq_ref，则先通过BKA算法批量提取key到join buffer，然后将buffer中的key作为参数传入MRR的调用接口。

 如果要打开MRR，需要设置以下两个优化器参数：

```
set optimizer_switch='mmr=on,mmr_cose_based=off';
```

 mmr参数控制是否打开，默认为on；mrr_cost_based控制是否根据优化器的计算成本来使用mrr特性，默认是on；如果希望尽可能使用mrr，可以设置为off。

 如果执行计划的Extra部分存在Using MRR，就是使用了该特性。

 BKA(Batch Key Access)是5.6引入的新算法,结合MRR特性进行高效join操作。

- 将外循环中相关的列放入Join Buffer中。
- 批量将Key(索引键值)发送到MRR接口
- MRR通过收到的Key，根据对应的Primary Key(RowID)进行排序，然后再根据排序后的Primary Key(RowID)顺序读取聚集特性，得到需要的列数据。
- 返回结果集给客户端。

 5.7以后, BKA默认是打开的，由优化器中的参数batched_key_access来控制。如果要使用BKA，则需要先打开MRR特性。

```
set optimizer_switch='mrr=on,mrr_cost_based=off,batch_key_access=on';
```

 判断是否使用了BKA算法，需要查看执行计划中extra部分是否含有”Using join buffer”字符串。

 BKA很多情况下可以提高廉洁效率，但是对Join有一定条件限制，一个条件时连接的列要求是唯一索引或普通索引，但是不能是主键；另一个是有对非主键列的查询操作，否则优化器就通过覆盖索引等方式直接得到需要的数据，而不需要回表。

### 15.4.6 优化嵌套查询

 4.1版本开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个查询结果作为过滤条件用在另一个查询中。有些情况下，可以用JOIN替代。

 连接之所以更有效率一些，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。

### 15.4.7 MySQL如何优化OR条件

 对于含有OR的查询子句，如果要利用索引，则OR之间的每个条件列都必须用到索引，如果没有索引，则应该考虑增加索引。

### 15.4.8 优化分页查询

 一般分页查询，通过创建覆盖索引能够比较好地提高性能。常见又头疼的场景是 limit 1000,20。 前1000条记录都会被抛弃，查询和排序的代价非常高

#### 1. 第一种优化思路

 在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

#### 2. 第二种优化思路

 把Limit转换成某个位置的查询，列入假设每页10条数据，和开发人员协商一下，反野过程中，通过增加一个参数 last_page_record,用来记录上一页最后一行的租赁编号rental_id，加入是15460，那么在翻页到42页时，可以根据最后一条记录向后追述。

```
select * from payment where rental_id < 15640 order by rental_id desc limit 10
```

注意：这种思路只会出现在排序字段不会出现重复值的特定环境。

### 15.4.9 使用SQL提示

 SQL提示是优化数据库的一个重要手段，简单地就是在SQL语句中增加一些人为的提示来达到优化操作的目的。

```
SELECT SQL_BUFFER_RESULSTS * FROM ...
```

 这个语句强制MySQL生成一个临时结果表，只要临时结果集生成后，所有表上的锁定均被释放，可以尽快释放锁资源。

 下面是常用的SQL提示。

#### 1. USE INDEX

 希望MySql去参考的索引列表

```
select count(*) from rental use index(idx_rental_date)
```

#### 2. IGNORE INDEX

 单纯想让MySQL忽略1个或者多个索引。

```
select count(*) from rental ignore index(idx_rental_date)
```

#### 3. FORCE INDEX

 强制MySQL使用一个特定的所有，可在查询中使用FORCE INDEX作为HINT。例如当不强制使用时，以为大部分库存inventory_id都是大于1，因此会默认进行全表扫描

```
select * from rental force index(idx_fk_inventory_id) where inventory_id > 1
```

## 15.5 直方图

 8.0引入的新功能。可以对一张表的一列做数据分布的统计，特别是针对没有索引的字段，可以帮助优化器找到更优的执行计划。主要场景就是用来计算字段选择性，即过滤效率。

### 15.5.1 什么是直方图

 在数据库中，查询优化器负责将SQL转换为最有效的执行计划。有时候一些字段分布不均匀，导致优化器对某些值不会选择最优的执行计划，使得效率低。为了能做出更准确的选择，优化器需要了解条件列中具体的数据分布情况，而直方图的引入就是为了统计这些信息。

 主要操作命令有以下两个：

- 生成直方图：

```
ANALYZE TABLE tbl_name update HISTOGRAM ON col_name[, col_name] with N BUCKETS;
```

- 删除直方图

```
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name[, col_name]
```

 比如过滤性别字段，执行计划中的filtered值都是50%，即优化器不知道数据实际分布情况，只是按照值的个数来进行平均分配，如果再gender上创建了直方图，则执行计划就会按照实际的数据分布进行过滤。

### 15.5.2 直方图的分类

 目前支持两种：等宽直方图(singleton)和登高直方图(equi-height)。共同点是，都将数据分到一系列的buckets中；区别在于如果列中不同值的个数小于buckets数，则为等宽直方图，反之则为登高直方图。会自动将数据滑到不同的bucket中，也会自动决定创建那种类型的直方图。

 直方图的统计信息存放在information_schema库的column_statistics视图中。

 具体每个值代表什么含义略，见原书252页。

### 15.5.3 直方图实例应用

 略

## 15.6 使用查询重写

 5.7中，提供了Query Rewrite Plugin，可以通过规则匹配的方式，将符合条件的SQL进行重写，从而达到调整执行计划或其他目标。

 在$mysqlhome/share目录下执行安装脚本，创建query_rewrite数据库和rewrite_rules规则表

```
mysql -uroot -p < install_rewrite.sql
mysql> show databases like 'query%'
mysql> use query_rewrite
mysql> show tables
mysql> SHOW GLOBAL VARIBALES LIKE 'rewriter_enabled'
```

 SQL重写插件安装之后，即使关闭插件，仍然会有一定的额外开销，考虑到插件可以动态安装和打开，因此如果不是确定要使用这一插件，没有必要重新安装

(1) 增加匹配规则，将”select ?”全部重写为 “select ? + 1”

```
INSERT INTO query_rewrite_rules(partten, replacement) values('select ?', 'select ? + 1');
```

(2) 刷新使规则生效

```
CALL query_rewrite.flush_rewrite_rules();
```

 注意，改写对函数无效

 在使用这一特性时，需要做好充分的测试。

## 15.7 常用SQL技巧

### 15.7.1 正则表达式的使用

[![img](https://pic.imgdb.cn/item/5f5a1334160a154a67c70de8.jpg)](https://pic.imgdb.cn/item/5f5a1334160a154a67c70de8.jpg)

```
select first_ name, email from customer where email regexp "@163[,.] com$";
```

### 15.7.2 巧用RAND()提取随机行

 ORDER BY rand() 能够把数据随机排序

### 15.7.3 利用 GROUP BY 的 WITH ROLLUP语句

 使用WITH ROLLUP语句可以检索出更多分组聚合信息。完成用户想要得到任何一个分组以及分组组合的聚合信息值。

 注意：当使用ROLLUP时，不能同时使用ORDER BY 子句进行结果排序。此外LIMIT用在ROLLUP后面。

### 15.7.4 用BIT GROUP FUNCTIONS做统计

 略

### 15.7.5 数据库名、表名大小写问题

 数据库每个表至少对应数据库目录中的一个或多个文件，所以操作系统大小写敏感决定了数据库名和表名大小写敏感。

 在MySQL中，如何在硬盘上保存、使用表名和数据库名是由lower_case_tables_name系统变量决定，用户可以在启动MySQL服务时设置这个系统变量

[![img](https://pic.imgdb.cn/item/5f5a1660160a154a67c871f9.jpg)](https://pic.imgdb.cn/item/5f5a1660160a154a67c871f9.jpg)

 注意：在UNIX中将lower_case_tables_name 设置为1并且重启mysqld之前，必须先将数据库名和表名转换成小写。

### 15.7.6 使用外键需要注意的问题

 InnoDB存储引擎支持对外部关键字约束条件的检查。

# 第16章 锁问题

## 16.1 MySQL锁概述

 不同引擎支持不同的锁机制。MyISAM和MEMORY采用表级锁，BDB采用页面锁，但也支持行锁，InnoDB既支持行级锁，也支持表级锁，但是默认是行级锁。

- 表级锁：开销小，加锁块；不会出现死锁；锁粒度打，锁冲突概率高；
- 行级锁：开销大，加锁慢；会出现死锁；锁粒度最小，发生锁冲突的概率最低，并发度最高；
- 页面锁：开销和加锁时间介于以上两者间，会出现死锁，锁粒度介于两者之间。

 重点介绍表锁和行锁

## 16.2 MyISAM的表锁

### 16.2.1 查询表级锁征用情况

 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上表锁定争用情况。

### 16.2.2 MySQL表级锁

 有两种模式：表共享读锁和表独占写锁。

 MySQL中表锁兼容性

|      | None | 读锁 | 写锁 |
| :--- | :--- | :--- | :--- |
| 读锁 | 是   | 是   | 否   |
| 写锁 | 是   | 否   | 否   |

 MyISAM的读操作，不会阻塞其他用户对同一表的读请求，但是会阻塞同意表的写请求；写请求会阻塞读写请求。

### 16.2.3 如果加表锁

 MyISAM在执行查询语句前，会自动给设计所有表加读锁；在执行更新操作前，会自动给涉及表加写锁。显示加锁只是为了说明为题，并非必须如此。

 给MyISAM表显示加锁，一般是为了在一定程度模拟事务操作，实现对某一时间点多个表的一致性读取。

```
Lock tables read local, order_detail read local;
select statement...
Unlock tables;
```

- 示例加了 local 选项，是为了满足MyISAM表并发插入的条件下，允许其他用户在表尾并发插入记录。
- 在LOCK TABLES显示加表锁时，必须同时取得所有涉及表的锁，并且不支持锁升级。加锁有，只能访问显示加锁的这些表，而不能访问其他表；同时加的读锁，只能执行查询操作。在自动加锁的情况下也是如此。这也正是MyISAM不会出现死锁的原因。

### 16.2.4 并发插入

 MyISAM存储引擎有一个系统变量concurrent_insert，专门用以控制其并发插入的行为

- 0：不允许并发阿插入；
- 1：如果表中没有空洞（中间没有被删除的行），MyISAM允许在一个进行读表的同事，另一个进行从表尾插入记录，是默认设置；
- 2：无论有没有空洞，都允许插入。

 可以利用这个特性来解决查询和插入的锁争用，设置为2，然后定期优化表。

### 16.2.5 MyISAM的锁调度

 MyISAM的读写互斥，是串行的。同时请求读写锁时，优先获得写锁。这是它不适合有大量的更新和查询操作应用的原因。可能会导致查询操作很难获得锁。不过可以来调节该行为

- 指定启动参数low_priority_updates,默认给读请求优先权利；
- 通过执行命令SET LOW_PRIORITY_UPDATE=1，使该连接发出更新请求优先级降低；
- 通过指定INSERT,UPDATE,DELETE语句的LOW_PRIORITY属性，降低该局域的优先级。

 另外，提供了一个这种办法来调节读写冲突，即给系统参数设置max_write_lock_count，设置一个合适的值，当一个表达到这个值后，MySQL暂时将写请求优先级降低，给读进程一些获得锁的。

 一些需要长时间的查询操作，也会使写进程”饿死”，因此避免长时间查询操作，不要总想用一条SELECT语句来解决问题。

## 16.3 InnoDB锁问题

 与MyISAM最大不同有两点，一是支持事务，而是行级锁。

### 16.3.1 背景知识

#### 1. 事务及其ACID属性

 事务是由一组SQL语句组成的逻辑处理单元，有4个属性，简称为ACID属性。

- 原子性(Atomicity)：事务是一个原子操作单元，其对数据的修改，要么全部执行，要么全部不执行；
- 一致性(Consistent)： 在事务开始和完成时，数据都必须保持一致状态。意味着所有相关的数据规则都必须应用于书屋的修改，以保持数据的完整性；事务结束时，所有的内部数据（比如B树或双向链表）也都必须是正确的。
- 隔离性(Isolation)：数据库系统提供一定的隔离机制，保证 事务不受外部并发操作影响的独立环境执行。
- 持久性(Durable)：事务完成之后，对数据的修改时永久性的，即使出现系统故障也能够保持。

#### 2. 并发事务带来的问题

- 更新丢失：当两个事务或多个事务选择同一行，然后基于最初选定的值更新改行，由于不知道互相存在，就会发生丢失更新问题。
- 脏读：一个事务正在对一条记录做修改，在这个事务完成并提交前，另外一个事务来读取同一条记录，如果不加控制，就读取了脏数据。
- 不可重复读：一个事务正在读取某个数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变或某些记录已经被删除了。
- 幻读(Phantom Read)： 一个事务按相同的查询条件重新查询已经检索过的数据，却发现其他事务插入了满足其查询条件的新数据。

#### 3. 事务隔离级别

 上面讲的问题，更新丢失通常应该是完全避免的，但防止丢失更新，并不能单靠数据库事务控制器来解决，需要应用程序对更新数据加必要的锁来解决。

 脏读、不可重复读、幻读，其实都输数据库读一致性问题，必须由数据库提供一定的事务隔离机制来解决，基本上分为两种。

- 一种是在读取数据前，加锁
- 一种是不加任何锁，通过一定机制胜一个数据请求时间点一致性数据快照，并且用这个快照提供一定级别的一致性读取。这个技术叫做数据多版本并发控制，也叫作多版本数据库。

 ISO/ANSI SQL92定义了4个事务隔离级别。

[![img](https://pic.imgdb.cn/item/5f5a3811160a154a67d4ff4a.jpg)](https://pic.imgdb.cn/item/5f5a3811160a154a67d4ff4a.jpg)

 各具体数据库不一定完全提供上述4个隔离级别，例如Oracle提供Read commited和Serializable两个标准隔离级别，另外还提供自己定义的Read Only隔离级别，SQL Server除了支持以上隔离级别，还支持一个叫做“快照”的隔离级别，是一个用MVCC实现的Serializable隔离级别。MySQL支持全部4个，但有一些MySQL自己的特点，比如在一些隔离级别下采用MVCC一致读，但某些情况下又不是，后面进一步介绍。

### 16.3.2 获取InnoDB行锁争用情况

```
show status like 'innodb_row_locks'
```

通过设置下面两个参数可以设置监视器，可以定期将包含锁冲突信息的日志写入ErrorLog

```
SET GLOBAL innodb_status_output=ON
SET GLOBAL innodb_status_out_locks=ON
```

也可以用以下语句来查看最新的状态信息

```
show engine innodb status
```

停止监视器

```
SET GLOBAL innodb_status_output=OFF
SET GLOBAL innodb_status_out_locks=OFF
```

### 16.3.3 InnoDB的行锁模式及加锁

- 共享锁(S)：允许一个事务读一行，阻止其他事务获得相同数据集的排他锁。
- 排他锁(X)：允许获得排它锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

 InnoDB还有两种内部使用的意向锁，都是表锁。

- 意向共享锁(IS)：事务打算给数据行家共享锁，事务再给一个数据行加共享锁前必须获得该表的IS锁。
- 意向排他锁(IX)：事务打算给数据行加行排他锁，事务在给一个数据库行家排他锁之前必须获得该锁。

[![img](https://pic.imgdb.cn/item/5f5c2f87160a154a6706af5f.jpg)](https://pic.imgdb.cn/item/5f5c2f87160a154a6706af5f.jpg)

 如果一个事务请求与当前的锁兼容，就成功获取锁，否则不兼容，就要等待锁释放。

 意向锁时InnoDB自动加的，不需要用户干预，对于UPDATE, DELETE, INSERT语句，InnoDB会自动加排他锁，但是普通的SELECT语句不会加任何锁，但是可以显示给数据集加排他锁或者共享锁。

```
5.7版本
共享锁：
SELECT * FROM TABLE_NAME WHERE ... LOCK IN SHARE MODE;
排他锁：
SELECT * FROM TABLE_NAME WHERE ... LOCK FOR UPDATE;

共享锁：
SELECT * FROM TABLE_NAME WHERE ... LOCK FOR SHARE;
排他锁：
SELECT * FROM TABLE_NAME WHERE ... LOCK FOR UPDATE [NOWAIT|SKIP LOCKED];
```

 5.7中，排他锁如果遇到锁等待默认等待50s，会造成高并发系统的连接数快速增加。为了解决问题，加了两个参数，顾名思义。

### 16.3.4 InnoDB行锁实现方式

 InnoDB的行锁是通过给索引上的索引项加锁来实现的，没有索引就通过隐藏的聚簇索引来进行加锁。

- Record Lock：对索引项进行加锁
- Gap lock：对索引项之间的”间隙”、第一条记录前的”间隙”或最后一条记录后的”间隙”加锁
- Next-Key lock：前两种的组合，对记录及其前面的间隙加锁。

 这意味着如果不通过索引条件检索数据，那么InnoDB将对表中所有记录加锁，实际效果跟表一样。可能会导致大量的锁冲突。

```
select * from tab_no_index where id=1 for update;
```

 但是会出现所等待，因为在没有索引的情况下，InnoDB会对所有记录都加锁。

(2) 由于行锁针对索引加的锁，不是针对记录加的锁，索然是访问不同的记录，但是如果使用相同索引键，会出现锁冲突。

(3)当表有多个索引，不同事物可以使用不同的索引锁定不同的行

(4)虽然使用了索引字段，但是执行计划不一定使用了索引，分析锁冲突时，别忘了检查SQL的执行计划。

### 16.3.5 Next-Key锁

 使用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引加锁；对于键值在条件范围内但不复存在的记录，叫做间隙，InnoDB也会对这个间隙加锁，这种锁机制叫Next-Key锁。

```
select * from emp where empid > 100 for update;
```

 不仅会对符合条件的empId的值加锁，也会对empId大于101的间隙加锁。

 一方面为了防止幻读，以满足相关隔离级别的要求。如果不加间隙锁，别的事务插入了满足条件的记录，就会发生幻读。另一方面，是为了满足其回复和复制的需要。

 特别要说明， 如果使用相同条件给一个不存在的记录加锁，也会使用Next-Key锁。

### 16.3.6 恢复和复制的需要，对InnoDB锁机制的影响

 MySQL通过BINLOG记录执行成功的INSERT, UPDATE, DELETE等更新数据的SQL语句，并由此实现MySQL数据库恢复和主从复制。5.7支持3种日志格式，基于语句的日志格式SBL、基于行的日志RBL和混合模式。还支持4种复制模式。

- 基于SQL语句的复制SBR, 最早的复制模式
- 基于行数据的复制RBR：5.1以后支持的模式，主要有点事支持对非安全SQL的复制
- 混合复制模式：对安全的SQL采用SBR，非安全的使用RBR
- 使用全局事务ID(GTIDs)的复制：主要是解决主从自动同步一致问题。

 对于基于日志格式(SBL)的恢复和复制，由于BINGLOG按照事务提交的先后顺序记录，因此要正确恢复或复制数据，就必须满足：在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，就是不允许出现幻读。

 两个事务都是简单的读source_tab表，相当于执行一个普通SELEC语句，用一致读就可以了。Oracle急救室这么做的，通过MVCC技术实现对版本数据来直线一致性读。但是InnoDB却给source_tab加了共享锁。

 原因是为了保证恢复和复制的正确性。不然其他事务进行了更新事务，就可能导致数据恢复的结果错误。

 设置系统变量inno_locks_unsafe_for_binlog的值为on后，则不再加锁。

 INSERT.. SELECT…和CREATE TABLE… SELECT..语句，会组织对源表的并发啊更新，如果查询比较复杂，还会造成严重的性能问题，避免使用，这是Unsafe SQL。

 如果应用中一定要用这种SQL来实现业务逻辑，又不希望对源表并发更新产生影响，可以采取以下3种措施。

- inno_locks_unsafe_for_binlog设置为on，代价是BINLOG可能无法正确恢复和复制数据，不推荐
- 通过使用”select * from source_tab… Into outfile”和”load data infile…”语句组合来间接实现，采用这种方式MySQL不会给source_tab加锁
- 使用基于行的BINLOG格式和基于行数据的复制

### 16.3.7 InnoDB在不同隔离级别下的一致性读及锁的差异

 在不同隔离级别下，InnoDB处理SQL时采用的一致性读策略和需要的锁是不同的。

[![img](https://pic.imgdb.cn/item/5f5c3b7c160a154a670964d3.jpg)](https://pic.imgdb.cn/item/5f5c3b7c160a154a670964d3.jpg)

 许多SQL，隔离级别越高，对记录集的锁就越严格，尤其是范围条件时。

 通过优化逻辑，大部分应用使用ReadCommited隔离级别就够了。

### 16.3.8 什么时候使用表锁

 个别特殊事务中，可以考虑使用表锁

- 事务需要更新大部分或全部数据
- 事务涉及到多个表，比较复杂，很可能引起死锁，造成大量事务回滚。

(1) LOCK TABLES索然可以加表级锁，但是表锁不是InnoDB存储引擎层管理的，是上一层MySQL Server负责的，仅当autocommit=0， innodb_table_locks=1(默认设置)，InnoDB层才知道MySQL加的表锁，否则InnoDB无法自动检测并处理这种死锁。

(2) 在LOCK TABLES对InnoDB表加锁时要注意，将AUTOCOMMIT设置为0，否则不会给表加锁；事务结束前，不要用UNLOCK TABLES释放锁，因为会隐式提交事务，所以先COMMIT在UNLOCK。

### 16.3.9 关于死锁

 设置合适的锁等待超时阈值，可以避免这种情况发生。

 大部分死锁都是设计问题，常用解决方法也是。

(1) 如果不同程序会并发取多个表，应尽量约定以相同顺序来访问表，这样可以大大降低产生死锁的机会。

(2) 批量处理数据时，如果实现对数据排序，保证每个线程安固定顺序处理记录，也可以大大降低出现死锁的概率。

(3) 在事务中，如果要更新记录，应该申请足够级别的锁，不应先申请共享锁，更新时再申请排他锁。

(4) 前面讲过，在REPREATABLE-READ隔离级别下，如果两个线程对相同条件加排他锁，没有符合该记录的情况下，两个线程都会加锁成功。隔离级别改成READ COMMITED，就可避免问题。

(5) 隔离级别为READ COMMITED时，如果两个线程都先执行SELECT…FOR UPDATE，判断是否存在符合条件的记录，如果没有，就插入记录。此时，只有一个线程能插入成功，另外一个线程会出现锁等待，当第一个线程提交后，第二个线程会因主键重出错，但虽然这个线程出错了，却会获得一个排他锁！如果有第三个又来申请排他锁，也会出现死锁。

 如果出现死锁，可以用SHOW INNODB STATUS命令来确定最后一个死锁产生的原因。

# 第17章 优化MySQL Server

## 17.1 MySQL体系结构概览

[![img](https://pic.imgdb.cn/item/5f5e114b160a154a679bc6e5.jpg)](https://pic.imgdb.cn/item/5f5e114b160a154a679bc6e5.jpg)

 默认情况下，MySQL有7组后台线程，分别是1个主线程、4组IO线程、1个锁线程和一个错误监控线程。5.5和5.6有分别新增一个purge线程和一个page cleaner线程。

- master thread：主要负责将脏缓存也刷新到数据文件，执行purge操作，触发检查点，合并插入缓冲区等。
- insert buffer thread：主要负责插入缓冲区的合并操作。
- read thread：负责数据库读取操作，可配置多个读线程。
- write thread：负责数据库写操作，可配置多个写线程。
- log thread：用于将重做日志刷新到logfile中。
- purge thread：5.5版本后单独的purge thread线程执行purge操作
- page cleaner thread：5.6之后，用来执行buffer pool中脏页的flush操作。
- lock thread：负责锁控制和死锁检测等。
- 错误监控线程：主要负责监控和错误处理

通过show engine innodb status命令查看这些线程的状态。

## 17.2 MySQL内存管理及优化

### 17.2.1 内存优化原则

- 尽量多的内存分配给MySQL做缓存，但要给操作系统和其他程序的运行留足够内存，否则如果产生SWAP页转换，将严重影响性能。
- MyISAM的数据文件读取依赖于操作系统自身的IO缓存，因此，如果有MyISAM表，就要预留更多的内存给操作系统做IO缓存。
- 排序区、连接区等缓存是分配给每个数据库会话专用的，其默认值的设置要根据最大连接数合理分配。如果设置太大，不但浪费内存资源，而且在并发连接较高时会导致物理内存耗尽。

### 17.2.2 MyISAM内存优化

 MyISAM存储引擎使用key buffer缓存索引块，可以加速MyISAM索引的读写速度。对于MyISAM表的数据块，MySQL没有特别的缓存机制，完全依赖操作系统的IO操作。

#### 1. key_buffer_size设置

 这个参数决定，直接影响MyISAM表的存取效率。对于一般MyISAM数据库，建议至少用1/4可用内存分配给该值。在参数文件设置该值

```
key_buffer_size = 4G
```

 可以检查key_read_requests, key_reads, key_write_requests和key_writes等MySQL状态变量来评估索引缓存效率。一般来说，索引块物理读比率key_reads/key_read_requests应小于0.01。索引块写比率key_writes/key_writes_request也应该更小。

 这和应用特点相关，如果是更新和删除特别多，key_writes/key_writes_request可能会接近1，而每次更新很多行的应用key_writes/key_writes_request就会比较小。

 除了通过索引块的物理写比率衡量key buffer的效率外，我们也可以通过评估key buffer的使用率来判断索引缓存设置是否合理。

 1 - (key_blocks_unused × key_cache_block_size / key_buffer_size)

 一般是使用率在80%左右比较合适。大于80%，可能因索引缓存不足而导致性能下降；小于80%会导致内存浪费。

#### 2. 使用多个索引缓存

 MySQL通过各session共享的key buffer提高了MyISAM索引存取的性能，但并不能消除session之间对key buffer的竞争。5.1版本开始引入多索引缓存机制，从而可以将不同表的索引缓存到不同的key buffer中。

```
#新建key buffer, hot_cache是新建索引缓存的名称，global代表对每一个新的连接都有效
set global hot_cache.key_buffer_size = 128*1024
# 删除
set global hot_cache.key_buffer_size = 0
```

 不能删除默认的key_buffer。

 默认情况下，MySQL使用key buffer缓存MyISAM的索引，可以通过case index命令指定表的索引缓存。

```
cache index sales, sales2 in hot_cache
```

 除了行数命令动态创建并分配辅助索引缓存外，更常见的做法是通过配置文件在MySQL启动时自动创建并加载所有缓存。

```
key_buffer_size = 4G
hot_cache.key_buffer_size = 2G
cold_cache.key_buffer_size = 1G
init_file=/path/to/data_directory/mysql_init.sql
```

在mysql_init.sql中，通过cache index命令分配索引缓存

#### 3. 调整”中间点插入策略”

 默认情况下，MySQL使用简单的LRU策略来选择要淘汰的索引数据块。这个算法不是很精细，某些情况会淘汰真正的热块。

 如果出现这种情况，除了使用上面介绍的多个索引缓存机制外，还可以利用重点插入策略来优化索引块淘汰算法。

 它把LRU链分成两部分，hot子表和warm子表，当索引块读入内存时，先被放到LRU链表重点，即warm子表的尾部，当达到命中一定的次数后，该索引块会被晋升到hot表的尾部；字后，该数据在host子表流转，如果其到达hot子表头部超过一定时间，它将由hot子表的头部降级到warm子表的头部；当需要淘汰索引块时，会优先淘汰warm表头部的内存块。

 听过调节key_cache_division_limit来控制多大的比例做warm表。

```
Set global key_cache_division_limit = 70
Set global hot_cache.key_cache_division_limit = 70;
```

 还可以调节key_cache_age_threshold控制数据库由hot子表向warm子表降级的时间，值越小，降级越快。对应有N个块的缓存索引来说，在最后N×key_cache_age_threshold/100次未命中，就会降级。

#### 4. 调整read_buffer_size和read_rnd_buffer_size

 如果需要经常顺序扫描MyISAM表，可以增大read_buffer_size来改善性能。read_buffer_size是每个session独占的，如果默认值太大，就会造成内存浪费。

 如果经常排序，适当增大read_rnd_buffer_size，也可以改善此类SQL的性能，也不要设置得太大。

### 17.2.3 InnoDB内存优化

 InnoDB的缓存机制和MyISAM不尽相同。

#### 1. InnoDB 缓存机制

 InnoDB用一块内存区做IO缓存池，该缓存池不仅用来缓存InnoDB的索引块，而且用来缓存InnoDB的数据库。

 在内部，InnoDB缓存池逻辑上由free list、flush list和LRU list组成。顾名思义，free list是空闲缓存块列表， flush list是需要刷新到磁盘的缓存块列表，而LRU list是InnoDB正在使用的缓存块，是InnoDB buffer list pool的核心。

 InnoDB使用的LRU算法与MyISAM的中点插入策略LRU算法很类似，大致原理是：将LRU list分为young sublist和old sublist，数据从磁盘读入时，会将该缓存块插入到LRU list的”中点”, 即old list的头部了经过一定时间的访问(由innodb_old_blocks_time系统参数决定)，该数据块将会由old sublist转移到young_sublist的头部，也就是整个LRU的头部；随时间推移，young_sublist和old_sublist中较少被访问的缓存快将从格子的头部向尾部一定；需要淘汰数据块时，优先从链表尾部淘汰。这种设计同样是为了防止偶尔被访问的索引块将访问频繁索引块淘汰。

 脏页的刷新在于FLUSH list和LRU list这两个链表中，LRU也存在可以刷新的脏页，这里是可以直接刷新的，默认BP(INNODB_BUFFER_POOL)中不存在可用的数据页的时候，会扫描LRU list尾部的innodb_lru_scan_depth个数据页(默认是1024)，进行相关刷新操作。从LRU list淘汰的数据，会立刻放入到free list中去。

 通过调整InnoDB buffer pool的大小、改变young sublist和old sublist的分配比例、控制脏缓存的刷新活动、使用多个InnoDB缓存池等方法来优化InnoDB的性能。

#### 2. innoDB_buffer_pool_size的设置

 正常情况下，innodb_buffer_pool_size的值越大，缓存命中率就越高，IO就减少，性能也提高。一个专用数据库服务器上，可以将80%的物理内存分配给它。要注意设置过大导致页交换。

 通过以下命令查看使用情况

```
mysqladmin --socket=/tmp/mysql_3307.sock ext|grep -i innodb_buffer_pool
```

 可用一下公式计算InnoDB缓存池命中率

(1-innodb_buffer_pool_reads/innodb_buffer_pool_read_requests)×100

MySql8.0, 增加了一个自动分配内存的参数 innodb_dedicated_server,如果只允许了一个mysql，将这个参数设置为1时，会根据物理内存大小自动分配。

**物理内存小于1GB**

innodb_buffer_pool_size=128MB

innodb_log_file_size=48MB

innodb_flush_method=O_DIRECT_NO_FSYNC(如果系统允许)

**物理内存1GB~4GB**

innodb_buffer_size=物理内存×0.5

innodb_log_file_size=128MB

innodb_flush_method=O_DIRECT_NO_FSYNC(如果系统允许)

**物理内存1GB~4GB**

innodb_buffer_size=物理内存×0.75

innodb_log_file_size=1024MB

innodb_flush_method=O_DIRECT_NO_FSYNC(如果系统允许)

5.6或5.7也可以参照以上规则

#### 3. 调整 old sublist大小

 由innodb_old_blocks_pct决定，取值范围是5~95，默认值是75(约3/4)，通过以下命令可以查看当前设置

```
show global varibles like '%innodb_old_blocks_pct%'
```

可以 根据 InnoDB Monitor 的 输出 信息 来 调整 innodb_ old_ blocks_ pct 的 值。 例如， 在 没有 较大 表 扫描 或 索引 扫描 的 情况下， 如果 young/ s 的 值 很低， 可能 就 需要 适当 增大 innodb_ old_ blocks_ pct 的 值 或 减小 innodb_ old_ blocks_ time 的 值。

#### 4. 调整innodb_old_blocks_time的设置

 决定了 缓存 数据 块 由 old sublist 转移 到 young sublist 的 快慢，

```
可以 根据 InnoDB Monitor 的 输出 信息 来 调整 innodb_ old_ blocks_ time 的 值。 在 进行 表 扫描 时， 如果 non- youngs/ s 很低， young/ s 很高， 就应 考虑 将 innodb_ old_ blocks_ time 适当 调 大， 以 防止 表 扫描 将 真正 的 热 数据 淘汰。 更 酷 的 是， 这个 值 可以 动态 设置， 如果 要 进行 大的 表 扫描 操作， 可以 很 方便 地 临时 做 调整。
```

#### 5. 调整缓存池数量，减少内部对缓存池的争用

InnoDB 的 缓存 系统 引入 了 innodb_ buffer_ pool_ instances 配置 参数， 对于 较大 的 缓存 池， 适当 增大 此 参数 的 值， 可以 降低 并发 导致 的 内部 缓存 访问 冲突， 改善 性能。

#### 6. 控制innodb buffer刷新，延长数据缓存时间，减缓磁盘I/O

 在 InnoDB 找 不到 干净 的 可用 缓存 页 或 检查点 被 触发 等 情况下， InnoDB 的 后台 线程 就会 开始 把“ 脏 的 缓存 页” 回 写到 磁盘 文件 中， 这个 过程 叫 缓存 刷新。

 我们 通常 都 希望 buffer pool 中的 数据 在 缓存 中 停留 的 时间 尽可能 长， 以备 重用， 从而 减少 磁盘 IO 的 次数。

 通常buffer pool刷新快慢取决于两个参数。

 一个 是 innodb_ max_ dirty_ pages_ pct， 它 控制 缓存 池 中 脏 页 的 最大 比例， 默认值 是 50%， 如果 脏 页 的 数量 达到 或 超过 该 值， InnoDB 的 后台 线程 将 开始 缓存 刷新。

 另一个 是 innodb_ io_ capacity， 它 代表 磁盘 系统 的 IO 能力， 其 值 在 一定程度 上代 表 磁盘 每秒 可 完成 I/ O 的 次数。 innodb_ io_ capacity 的 默认值 是 200， 对于 转速 较低 的 磁盘， 如 7200RPM 的 磁盘， 可将 innodb_ io_ capacity 的 值 降低 到 100， 而对 于 固态 硬盘 和 由 多个 磁盘 组成 的 盘 阵， innodb_ io_ capacity 的 值 可以 适当 增大。

 innodb_ io_ capacity 决定 一批 刷新 脏 页 的 数量， 当 缓存 池 脏 页 的 比例 达到 innodb_ max_ dirty_ pages_ pct 时， InnoDB 大约 将 innodb_ io_ capacity 个 已 改变 的 缓存 页 刷新 到 磁盘； 当 脏 页 比例 小于 innodb_ max_ dirty_ pages_ pct 时， 如果 innodb_ adaptive_ flushing 的 设置 为 true， InnoDB 将 根据 函数 buf_ flush_ get_ desired_ flush_ rate 返回 的 重做 日志 产生 速度 来 确定 要 刷 新的 脏 页数。 在 合并 插入 缓存 时， InnoDB 每次 合并 的 页数 是 0. 05 × innodb_ io_ capacity。

 可以 根据 一些 InnoDB Monitor 的 值 来 调整 innodb_ max_ dirty_ pages_ pct 和 innodb_ io_ capacity。 例如， 若 innodb_ buffer_ pool_ wait_ free 的 值 增长 较快， 则 说明 InnoDB 经常 在等 待 空闲 缓存 页， 如果 无法 增大 缓存 池， 那么 应将 innodb_ max_ dirty_ pages_ pct 的 值 调 小， 或将 innodb_ io_ capacity 的 值 提高， 以 加快 脏 页 的 刷新。

#### 7. InnoDB doublewrite

 进行脏页刷新，采用双写策略。原因是：数据页大小(一般是16KB)与操作系统的IO数据也大小(一般是4KB)不一致，无法保证InnoDB缓存页被完整、一致地刷新到磁盘，而InnoDB的redo日志只记录了数据页改变部分，没有记录数据页的完整前像，发生部分写或断裂写时，就会出现无法恢复的问题，所以引入了该技术。

 实现 原理 是： 用 系统 表 空间 中的 一块 连续 磁盘 空间（ 100 个 连续 数据 页， 大小 为 2MB） 作为 doublewrite buffer， 当 进行 脏 页 刷新 时， 首先 将 脏 页 的 副本 写到 系统 表 空间 的 doublewrite buffer 中， 然后 调用 fsync() 刷新 操作系统 IO 缓存， 确保 副本 被 真正 写入 磁盘， 最后 InnoDB 后台 IO 线程 将 脏 页 刷新 到 磁盘 数据 文件。

[![img](https://pic.imgdb.cn/item/5f60af1f160a154a673678de.jpg)](https://pic.imgdb.cn/item/5f60af1f160a154a673678de.jpg)

 在做恢复时，如果发现不一致的页，就会用表空间的 double write buffer的相应副本来恢复数据页。

 默认是开启的

```
show global varibales like '%doublewrites%'
```

 对性能影响不太大，又能容忍极端情况下少量数据丢失的应用，可以配置文件添加innodb_doublewrite=0的参数来关闭doublewrite。

### 17.2.4 调整用户服务线程排序缓存区

 如果 通过 show global status 看到 sort_ merge_ passes 的 值 很大， 可以 考虑 通过 调整 参数 sort_ buffer_ size 的 值 来 增大 排序 缓存 区， 以 改善 带有 order by 子句 或 group 子句 SQL 的 性能。

 对于 无法 通过 索引 进行 连接 操作 的 查询， 可以 尝试 通过 增大， join_ buffer_ size 的 值 来 改善 性能。

 要注意以上两个都是面向客户服务线程分配，太大会造成内存浪费，尤其是join buffer.最好策略就是设置较少的全局join_buffer_size, 复杂连接操作的session设置较大的。

## 17.3 InnoDB log机制及优化

 InnoDB采用redo log来保证事务更新的一致性和持久性。

### 17.3.1 InnoDB重做日志

[![img](https://pic.imgdb.cn/item/5f60b375160a154a673789a5.jpg)](https://pic.imgdb.cn/item/5f60b375160a154a673789a5.jpg)

 更新操作是，操作流程大致是：

(1) 将数据读入InnoDB buffer pool，并对相关记录加独占锁；

(2) 将UNDO信息写入undo表空间的回滚段中；

(3) 更改缓存页中的数据，将更新记录写入redo buffer中；

(4) 提交时，根据innodb_flush_log_at_trx_commit的设置，不同的方式将redo buffer 中的 更新 记录 刷新 到 InnoDB redo log file 中， 然后 释放 独占 锁；

（5） 最后， 后台 IO 线程 根据 需要 择 机 将 缓存 中 更新 过 的 数据 刷新 到 磁盘 文件 中。

 通过show engin innodb status命令查看当前日志的写入情况。

[![img](https://pic.imgdb.cn/item/5f60b4af160a154a6737d9f9.jpg)](https://pic.imgdb.cn/item/5f60b4af160a154a6737d9f9.jpg)

 LSN(Log Sequence Number)称为日志序列号，它实际上对应日志文件的偏移量，生成公式是：

 新的LSN=旧的LSN+写入日志大小。

 例如， 日志 文件 大小 为 600MB， 目前 的 LSN 是 1GB， 现在 要将 512 字节 的 更新 记录 写入 redo log， 则 实际 写入 过程 如下。

 求出 偏移量： 由于 LSN 数值 远大 于 日志 文件 大小， 因此 通过 取 余 方式， 得到 偏移量 为 400MB。 写入 日志： 找到 偏移 400MB 的 位置， 写入 512 字节 日志 内容， 下一个 事务 的 LSN 就是 1000000512。 由 以上 介绍 可知， 除 InnoDB buffer pool 外， InnoDB log buffer 的 大小、 redo 日志 文件 的 大小 以及 innodb_ flush_ log_ at_ trx_ commit 参数 的 设置 等， 都会 影响 InnoDB 的 性能。 下面 我们 就 介绍 这 几个 参数 的 优化 调整。

### 17.3.2 innodb_flush_log_at_trx_commit的设置

 innodb_ flush_ log_ at_ trx_ commit 参数 可以 控制 将 redo buffer 中的 更新 记录 写入 到 日志 文件 以及 将 日志 文件 数据 刷新 到 磁盘 的 操作 时机。 通过 调整 这个 参数， 可以 在 性能 和数 据 安全 之间 做 取舍。

 如果 这个 参数 设置 为 0， 在 事务 提交 时， InnoDB 不会 立即 触发 将 缓存 日志 写到 磁盘 文件 的 操作， 而是 每秒 触发 一次 缓存 日志 回 写 磁盘 操作， 并 调用 操作系统 fsync 刷新 IO 缓存。

 如果 这个 参数 设置 为 1， 在 每个 事务 提交 时， InnoDB 立即 将 缓存 中的 redo 日志 回 写到 日志 文件，并 调用 操作系统 fsync 刷新 IO 缓存。

 如果 这个 参数 设置 为 2， 在 每个 事务 提交 时， InnoDB 立即 将 缓存 中的 redo 日志 回 写到 日志 文件， 但 并不 马上 调用 fsync 来 刷新 IO 缓存， 而是 每秒 只做 一次 磁盘 IO 缓存 刷新 操作。

 默认是1，最安全，但是有较大性能损失。

 0和2都能明显减少日志同步IO，加快事务提交。

 0时，如果数据库崩溃，最后一秒的事务重做日志会丢失，最不安全，但效率最高

 设置为2，如果数据库崩溃，由于已执行重做日志写入磁盘操作，只是没有做磁盘IO刷新操作。比较折中，主要操作系统不崩溃，就不会丢失数据

### 17.3.3 设置log file size，控制检查点。

 一个日志文件写满，会自动切换到另一个日志文件，但切换时会触发数据库检查点，导致缓存脏页小批量刷新，会明显降低InnoDB性能。

 一般来说，平均每半小时写满一个日志文件比较合适。

[![img](https://pic.imgdb.cn/item/5f60bac9160a154a6739720d.jpg)](https://pic.imgdb.cn/item/5f60bac9160a154a6739720d.jpg)

[![img](https://pic.imgdb.cn/item/5f60baec160a154a67398178.jpg)](https://pic.imgdb.cn/item/5f60baec160a154a67398178.jpg)

[![img](https://pic.imgdb.cn/item/5f60baf8160a154a6739870e.jpg)](https://pic.imgdb.cn/item/5f60baf8160a154a6739870e.jpg)

### 17.3.4 调整Innodb_log_buffer_size

 innodb_log_buffer_size决定InnoDB重做日志缓存池的大小，默认是16MB。对于 可能 产生 大量 更新 记录 的 大 事务， 增加 innodb_ log_ buffer_ size 的 大小， 可以避免 InnoDB 在 事务 提交 前 就 执行 不必 要的 日志 写入 磁盘 操作。 因此， 对于 会在 一个 事务 中 更新、 插入 或 删除 大量 记录 的 应用， 我们 可以 通过 增大 innodb_ log_ buffer_ size 来 减少 日志 写 磁盘 操作， 从而 提高 事务处理 的 性能。

## 17.4 调整MySQL并发相关参数

### 17.4.1 调整max_connetions提高并发数。

 参数 max_ connections 控制 允许 连接 到 MySQL 数据库 的 最大 数量， 默认值 是 151。 如果 状态 变量 connection_ errors_ max_ connections 不 为零， 并且 一直 在 增长， 就说 明 不断 有 连接 请求 因 数据库 连接 数 已达 到 最大 允许 的 值 而 失败， 应考 虑 增大 max_ connections 的 值。

 在 Linux 平台 下， MySQL 支持 500 ～ 1000 个 连接 不是 难事， 如果 内存 足够、 不考虑 响应 时间， 甚至 能达到 上万 个 连接。 而在 Windows 平台 下， 受 其所 用 线程 库 的 影响， 最大 连接 数 有 以下 限制：

[![img](https://pic.imgdb.cn/item/5f60bbac160a154a6739c002.jpg)](https://pic.imgdb.cn/item/5f60bbac160a154a6739c002.jpg)

 每一个 session 操作 MySQL 数据库 表 都 需要 占用 文件 描述 符， 数据库 连接 本身 也要 占用 文件 描述 符， 因此， 在 增大 max_ connections 时， 也要 注意 评估 open- files- limit 的 设置 是否 够用。

### 17.4.2 调整back_log

 back_ log 参数 控制 MySQL 监听 TCP 端口 时 设置 的 积压 请求 栈 大小， 5. 6. 6 版本 以前 的 默认值 是 50， 5. 6. 6 版本 以后 的 默认值 是 50+（ max_ connections / 5）， 但 最大 不能 超过 900。 如果 需要 数据库 在 较短 时间 内 处理 大量 连接 请求， 可以 考虑 适当 增大 back_ log 的 值。

### 17.4.3 调整table_open_cache

[![img](https://pic.imgdb.cn/item/5f60bc01160a154a6739daac.jpg)](https://pic.imgdb.cn/item/5f60bc01160a154a6739daac.jpg)

### 17.4.4 调整 thread_cache_size

 为 加快 连接 数据库 的 速度， MySQL 会 缓存 一定 数量 的 客户 服务 线程 以备 重用， 通过 参数 thread_ cache_ size 可 控制 MySQL 缓存 客户 服务 线程 的 数量。 可以 通过 计算 线程 cache 的 失 效率 threads_ created/ connections 来 衡量 thread_ cache_ size 的 设置 是否 合适。 该 值 越 接近 1， 说明 线程 cache 命中率 越 低， 应考 虑 适当 增加 thread_ cache_ size 的 值。

### 17.4.5 innodb_lock_wait_timeout的设置

参数 innodb_ lock_ wait_ timeout 可以 控制 InnoDB 事务 等待 行 锁 的 时间， 默认值 是 50ms， 可以 根据 需要 动态 设置。 对于 需要 快速 反馈 的 交互式 OLTP 应用， 可以 将 行 锁 等待 超时 时间 调 小， 以 避免 事务长 时间 挂起； 对于 后台 运行 的 批处理 操作， 可以 将 行 锁 等待 超时 时间 调 大， 以 避免 发生 大的 回 滚 操作。

## 17.5 持久化变量

 8.0之前，一般都是通过set global命令来设置MySQL参数，一旦数据库实例重启，那么就会从配置文件中重新加载就得配置。

 8.0中，提供了全新的set persist/set persist only命令，可以对数据库参数做持久化的修改，通过set persist方式做过的修改，数据库实例在重启后，就能够正确加载修改后的参数。其中set persist命令会同时修改当前环境并持久化参数修改，set persist only不修改当前环境，仅仅对修改做持久化。

 可以通过reset persist命令来清除mysqld-auto.cnf文件中的所有配置，也可以通过reset persist + 参数名的方式，来清除某个制定的配置。

## 17.6 使用资源组

 8.0中，新增了资源组特性。通过资源组可以限制某个任务或查询对系统资源的使用。目前在8.0中，能够调整的是对CPU资源的使用限制和线程优先级。

 CPU层面可以控制的粒度是逻辑CPU个数。可以通过命令more/proc/cpuinfo | grep processor | wc -l 来查看服务器总攻的vcpu个数。

 在线程优先级层面，默认都是0，数字越小越高。系统资源组可以直接设置在-20~0之间；用户资源组可以在0到19之间。

 查看资源组

```
select * from information_schema.resource_groups;
```

 默认创建两个资源组，使用全部vcpu，线程优先级是0，属性不可修改，新创建的线程默认分配到两个中的一个。

 其他内容暂略。

# 第18章 磁盘I/O问题

## 18.1 使用固态硬盘

 固态硬盘是近年提升数据库I/O最常见、性价比最高的手段之一，应该优先考虑。

 传统机械硬盘受以下3个方面的影响：

- 盘片转速：相对来说，转速越高的磁盘读写速度也越快，但是受到机械性能和发热的限制，转速为4200到15000RPM
- 存储密度：密度更大在同样转速下，磁头每秒能够读写的数据量也更大
- 接口类型：一般有SATA2, SATA3, SAS等类型，速度不一样。

 性能上的最大瓶颈来自于随机I/O。

 固态硬盘由主控芯片加闪存芯片组成，不存在机械操作过程：

- 读写速度快：数倍，随机I/O可以达到数十倍，采用PCI-E接口还会更高
- 可高兴高：不存在易损坏的机械结构，相对来说更加稳定可靠
- 功耗低、体积小、重量轻、无噪音。

 有一些注意的方面：

- 合理规划成本
- 选择合适的幸好
- 合理预留空间

## 18.2 使用磁盘整列

 RAID(Redundant Array of Independent Disks)，独立磁盘冗余阵列，通常叫做磁盘阵列。

### 18.2.1 常见RAID级别及其特性

[![img](https://pic.imgdb.cn/item/5f61ff44160a154a6782259e.jpg)](https://pic.imgdb.cn/item/5f61ff44160a154a6782259e.jpg)

| RAID级别 | 特性                                                         | 优点                                                         | 缺点                                                        |
| :------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- |
| RAID6    | 在RAID5基础上，进一步加强数据保护而设计的一种RAID方式，实际是一种扩展RAID5等级。与RAID5不同之处在于处了每个硬盘上都有XOR校验区外，还有一个针对每个数据块的XOR校验区 | 每个数据块有了两个校验保护屏障（一个是分层校验，一个是总体校验），因此RAID6的数据冗余性能更好，每个RAID允许两块磁盘通识出现故障 | 写性能相对比RAID5更差：需要消耗两块硬盘的容量作为数据校验盘 |

### 18.2.2 如何选择RAID级别

- 数据读写都很频繁，可靠性要求也很高，最好选择RAID10;
- 数据读很频繁，写相对较少，对可靠性有一定要求，可以选择RAID5或RAID6;
- 读写都很频繁，但可以接受全部数据丢失，可以选择RAID0

## 18.3 虚拟文件卷或软RAID

 最初， RAID 都是 由 硬件 实现 的， 要 使用 RAID， 至少 需要 有一个 RAID 卡。 但 现在， 一些 操作系统 中 提供 的 软件包， 也 模拟 实现 了 一些 RAID 的 特性， 虽然 性能 上 不如 硬 RAID， 但 相比 单个 磁盘， 性能 和 可靠性 都 有所改善。

 在 不具备 硬件 条件 的 情况下， 可以 考虑 使用 上述 虚拟 文件 卷 或 软 RAID 技术， 具体 配置 方法 可 参见 Linux 帮助 文档。

## 18.4 使用Symbolic分布I/O

 MySQL 的 数据库 名 和 表 名 是与 文件 系统 的 目录 名 和 文件名 对应 的， 默认 情况下， 创建 的 数据库 和 表 都存 放在 参数 datadir 定义 的 目录 下。

 就可以 利用 操作系统 的 符号 连接（ Symbolic Links） 将 不同 的 数据库、 表 或 索引 指向 不同 的 物理 磁盘， 从而 达到 分布 磁盘 I/ O 的 目的。

[![img](https://pic.imgdb.cn/item/5f62093e160a154a6784c694.jpg)](https://pic.imgdb.cn/item/5f62093e160a154a6784c694.jpg)

## 18.5 禁止操作系统更新文件的atime属性

 它是一个系统下的一个文件属性，每当读取文件，就会将读操作发生的时间会写到磁盘上。

（1） 修改 文件 系统 配置文件/ etc/ fstab， 指定 noatime 选项：

```
LABEL=/ home /home ext3 noatime 1 2
```

（2） 重新 mount 文件 系统：

```
#mount -oremount /home
```

完成 上述 操作， 以后 读/ home 下文 件 就不 会 再写 磁盘 了。

## 18.6 调整I/O调度算法

传统磁盘读取数据分为以下3步：

1. 将磁头异动到磁盘表面的正确位置，寻道时间。
2. 等待磁盘旋转，需要的数据会异动到磁头下面，花费时间取决于磁盘转速
3. 磁盘继续旋转，知道所有需要的数据都经过磁头。

 很大程度上取决于寻到时间。为了减少时间，操作系统不对每次IO请求都直接寻道处理，而是将请求放入队列，然后合并和排序处理，减少磁盘寻道操作次数。

 为此 Linux 实现 了 4 种 I/ O 调度 算法， 分别 是 NOOP 算法（ No Operation）、 最后 期限 算法（ Deadline）、 完全 公平 队列 算法（ CFQ） 以及 预期 算法（ Anticipatory）。

[![img](https://pic.imgdb.cn/item/5f620b1e160a154a6785760d.jpg)](https://pic.imgdb.cn/item/5f620b1e160a154a6785760d.jpg)

 从上 面的 算法 中 可以 看到， 在 不同 的 场景 下 选择 不同 的 I/ O 调度 器 是 十分必要 的。 在 完全 随机 的 访问 环境 下， CFQ 和 Deadline 性能 差异 很小， 但是 在 有大 的 连续 I/ O 出现 的 情况下， CFQ 可能 会 造成 小 I/ O 的 响应 延时 增加， 所以 建议 MySQL 数据库 环境 设置 为 Deadline 算法， 这样 更稳 定。 对于 SSD 等 设备， 采用 NOOP 或者 Deadline 通常 也可以 获取 比 默认 调度 器 更好 的 性能。



```
#查看当前系统支持的IO调度算法
dmesg | grep -i scheduler
#查看当前设备(/dev/sda)使用的IO调度算法
more /sys/block/sda/queue/scheduler
#修改当前块设备(/dev/sda)使用的IO调度算法，修改IO调度算法后直接生效
echo "deadline" > /sys/block/sda/queue/scheduler
#永久修改IO调度算法，可以通过修改内核引导参数，增加elevator=调度程序名
vi /boot/grub/menu.list
#更改内容
kernel /boot/vmlinuz-2.6.18-308.e15 ro root=LABEL=/ elevator=deadline
```

## 18.7 RAID卡电池放点问题。

### 18.7.1 什么是RAID卡电池充放电

 RAID 卡 都有 写 缓存（ Battery Backed Write Cache）， 写 缓存 对 I/ O 性能 的 提升 非常 明显， 为了 避 免掉 电 丢失 写 缓存 中的 数据， 所以 RAID 卡 都有 电池（ Battery Backup Unit， 简称 BBU） 来 提供 掉 电 后 将 写 缓存 中的 数据 写入 磁盘。

 通俗 的 说， RAID 卡 电池 会 定期 充 放电， 定期 充 放电 的 操作 叫做 电池 Relearn 或者 电池 校准。

 略

### 18.7.2 RAID卡缓存策略

 略

### 18.7.3 如何应对RAID卡电池充放电带来的I/O波动

 略

## 18.8 NUMA架构

 略

# 第19章 应用优化

## 19.1 优化数据表的设计

### 19.1.1 优化数据表的类型

 使用函数对当前表进行分析，提出优化建议。

```
SELECT * FROM TBL_NAME PROCEDURE ANALYSE();
SELECT * FROM TBL_NAME PROCEDURE ANALYSE(16, 256)
```

第二个语句告诉不要为那些包含多余16个或256字节的ENUM类型提出建议，不然输出信息可能会很长。

### 19.1.2 通过拆分提高表的访问效率

(1) 垂直拆分，把主键和一些列放在一个表，主键和另外的一些列放在另外的表

(2) 水平拆分，放到多个不同的独立表或分区中。

水平拆分通常在一下几种情况使用：

- 表很大，分割后可以降低在查询时需要读的数据和索引的页数，通识也降低了索引的层数，提高查询速度。
- 表中的数据本来就有独立性，例如，分别记录各个地区的数据或不同时期的数据
- 需要把数据存放到多个介质上

 用分表的方式做水平拆分会给应用增加复杂度，通常在查询时需要多个表名，查询所有数据需要UNION操作。如果数据量不是特别大，可以优先采用分区的方式来做水平拆分。

### 19.1.3 逆规范化

- 增加冗余列：避免连接操作
- 增加派生列：避免连接操作，避免使用集函数。
- 重新组表：如果有许多用户要查看两个表连接出来的结果集，则把两个表重新组成一个表来减少连接而提高性能
- 分割表：

 需要一定的管理来维护数据完整性

- 批处理维护是对冗余列或派生列的修改积累到一定时间后，运行一批处理作用后存储过程对辅助或派生列进行修改，只能对实时性要求不高的情况下使用
- 由应用逻辑来实现：要求在同一事务对所有涉及表进行增删改差操作。
- 使用触发器。

## 19.2 数据库应用优化

### 19.2.1 使用连接池

 略

### 19.2.2 减少对MySQL的访问

#### 1.避免对同一数据做重复查询

#### 2.增加cache层

### 19.2.3 负载均衡

 利用某种均衡算法，将固定的负载量分布到不同的服务器上。

#### 1. 利用MySQL复制分流查询操作

 利用主从复制(具体见30章)可以有效分离更新和查询操作。

#### 2. 采用分布式数据库架构

略