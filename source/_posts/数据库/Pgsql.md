```sql
create database test03 encoding 'UTF8' lc_collate 'en_US.utf8' lc_ctype 'en_US.utf8'  template template0;


CREATE TABLE weather (
	city varchar(80),
	temp_lo int, -- 最低温度
	temp_hi int, -- 最高温度
	prcp real, -- 湿度
	date date
);
DROP TABLE weather;
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');

# 视图
CREATE VIEW myview AS
SELECT city, temp_lo, temp_hi, prcp, date, location
FROM weather, cities
WHERE city = name;

# 外键
CREATE TABLE weather (
city varchar(80) references cities(city),
temp_lo int,
temp_hi int,
prcp real,
date date
);

# 事务
BEGIN;
UPDATE accounts SET balance = balance - 100.00
WHERE name = 'Alice';
-- etc etc
COMMIT;

# 保存点
BEGIN;
UPDATE accounts SET balance = balance - 100.00
WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
WHERE name = 'Wally';
COMMIT;

# 窗口函数
# 下面是一个例子用于展示如何将每一个员工的薪水与他/她所在部门的平均薪水进行比较：
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM
empsalary;

# 我们可以通过OVER上的ORDER BY控制窗口函数处理行的顺序（窗口的ORDER BY并不一
# 定要符合行输出的顺序。）。下面是一个例子：
SELECT depname, empno, salary,
rank() OVER (PARTITION BY depname ORDER BY salary DESC) FROM empsalary;
```





```sql
SELECT salary, sum(salary) OVER () FROM empsalary;
```



由于在OVER子句中没有ORDER BY，窗口帧和分区一样，而如果缺
少PARTITION BY则和整个表一样。换句话说，每个合计都会在整个表上进行，这样我们为
每一个输出行得到的都是相同的结果。但是如果我们加上一个ORDER BY子句，我们会得到
非常不同的结果：



```sql
# 这里的合计是从第一个（最低的）薪水一直到当前行，包括任何与当前行相同的行（注意相
# 同薪水行的结果）。
SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;
salary | sum
--------+-------
3500 | 3500
3900 | 7400
4200 | 11600
4500 | 16100
4800 | 25700
4800 | 25700
5000 | 30700
5200 | 41100
5200 | 41100
6000 | 47100
```

# 继承

在PostgreSQL中，一个表可以从0个或者多个表继承。

```java
CREATE TABLE cities (
name text,
population real,
altitude int -- (in ft)
);
CREATE TABLE capitals (
state char(2)
) INHERITS (cities);
```

# SQL

## unicode

`U&'d\0061t\+000061'`

如果想要一个不是反斜线的转义字符，可以在字符串之后使用UESCAPE子句来指定，例
如：
`U&'d!0061t!+000061'UESCAPE'!'`

## 美元引用的字符串常量

注意在美元引用字符串中，单引号可以在不被转义的情况下使用。事实上，在一个美元引用
字符串中不需要对字符进行转义：字符串内容总是按其字面意思写出。

```
$$Dianne's horse$$
```

## 其他类型常量

一种任意类型的一个常量可以使用下列记号中的任意一种输入：
type 'string'
'string'::type
CAST ( 'string' AS type )

## 注释

一段注释是以双斜线开始并且延伸到行结尾的一个字符序列，例如：
`-- This is a standard SQL comment`
另外，也可以使用 C 风格注释块：

```
/* multiline comment
* with nesting: /* nested block comment */
*/
```

## 操作优先级

![](https://pic.imgdb.cn/item/625f55e0239250f7c5facff5.jpg)

## 下标

如果一个表达式得到了一个数组类型的值，那么可以抽取出该数组值的一个特定元素：
`expression[subscript]`
或者抽取出多个相邻元素（一个“数组切片”）：
`expression[lower_subscript:upper_subscript]`

## 操作符调用

对于一次操作符调用，有三种可能的语法：
`expression operator expression`（二元中缀操作符）
`operator expression`（一元前缀操作符）
`expression operator`（一元后缀操作符）
其中operator记号遵循第 4.1.3 节的语法规则，或者是关键词AND、OR和NOT之一，或者是
一个如下形式的受限定操作符名：
`OPERATOR(schema.operatorname)`
哪个特定操作符存在以及它们是一元的还是二元的取决于由系统或用户定义的那些操作
符。第 9 章描述了内建操作符。



```
SELECT
count(*) AS unfiltered,
count(*) FILTER (WHERE i < 5) AS filtered
FROM generate_series(1,10) AS s(i);
```

## 类型转换

```sql
CAST ( expression AS type )
expression::type
```

## 数组构造器

```sql
SELECT ARRAY[1,2,3+4];

SELECT ARRAY[1,2,22.7]::integer[];
```

## 行构造器

当在列表中有超过一个表达式时，关键词ROW是可选的。

```sql
SELECT ROW(1,2.5,'this is a test');
```

# 数据定义

## 默认值

```sql
CREATE TABLE products (
product_no integer,
name text,
price numeric DEFAULT 9.99
);

# 序列号
CREATE TABLE products (
product_no integer DEFAULT nextval('products_product_no_seq'),
...
);

# 这里nextval()函数从一个序列对象第 9.16 节）。还有一种特别的速写：
CREATE TABLE products (
product_no SERIAL,
...
);

# 检查约束
CREATE TABLE products (
product_no integer,
name text,
price numeric CHECK (price > 0)
);
# 我们也可以给与约束一个独立的名称。这会使得错误消息更为清晰，同时也允许我们在需要
# 更改约束时能引用它。语法为：
CREATE TABLE products (
product_no integer,
name text,
price numeric CONSTRAINT positive_price CHECK (price > 0)
);

#一个检查约束也可以引用多个列。例如我们存储一个普通价格和一个打折后的价格，而我们
# 希望保证打折后的价格低于普通价格：
CREATE TABLE products (
product_no integer,
name text,
price numeric CHECK (price > 0),
discounted_price numeric CHECK (discounted_price > 0),
CHECK (price > discounted_price)
);

# 表约束也可以用列约束相同的方法来指定名称：
CREATE TABLE products (
product_no integer,
name text,
price numeric,
CHECK (price > 0),
discounted_price numeric,
CHECK (discounted_price > 0),
CONSTRAINT valid_discount CHECK (price > discounted_price)
);
```

## 非空约束

```sql
CREATE TABLE products (
product_no integer NOT NULL,
name text NOT NULL,
price numeric
);
```

NOT NULL约束有一个相反的情况：NULL约束。这并不意味着该列必须为空，进而肯定是
无用的。相反，它仅仅选择了列可能为空的默认行为。SQL标准中并不存在NULL约束，因此
它不能被用于可移植的应用中（PostgreSQL中加入它是为了和某些其他数据库系统兼容）。

但是某些用户喜欢它，因为它使得在一个脚本文件中可以很容易的进行约束切换。例如，初
始时我们可以：

```sql
CREATE TABLE products (
product_no integer NULL,
name text NULL,
price numeric NULL
);
```

## 唯一约束

```sql
CREATE TABLE products (
	product_no integer UNIQUE,
	name text,
	price numeric
);

CREATE TABLE products (
	product_no integer,
	name text,
	price numeric,
	UNIQUE (product_no)
);

CREATE TABLE example (
	a integer,
	b integer,
	c integer,
	UNIQUE (a, c)
);

# 我们可以通常的方式为一个唯一索引命名：
CREATE TABLE products (
product_no integer CONSTRAINT must_be_different UNIQUE,
name text,
price numeric
);

```

## 主键

```sql
CREATE TABLE products (
product_no integer UNIQUE NOT NULL,
name text,
price numeric
);
CREATE TABLE products (
product_no integer PRIMARY KEY,
name text,
price numeric
);

CREATE TABLE example (
a integer,
b integer,
c integer,
PRIMARY KEY (a, c)
);
```

## 外键

```sql
CREATE TABLE orders (
order_id integer PRIMARY KEY,
product_no integer REFERENCES products (product_no),
quantity integer
);

CREATE TABLE t1 (
a integer PRIMARY KEY,
b integer,
c integer,
FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```

​	我们知道外键不允许创建与任何产品都不相关的订单。但如果一个产品在一个引用它的订单
创建之后被移除会发生什么？SQL允许我们处理这种情况。直观上，我们有几种选项：

```sql
CREATE TABLE order_items (
product_no integer REFERENCES products ON DELETE RESTRICT,
order_id integer REFERENCES orders ON DELETE CASCADE,
quantity integer,
PRIMARY KEY (product_no, order_id)
);
```

​	限制删除或者级联删除是两种最常见的选项。

​	与ON DELETE相似，同样有ON UPDATE可以用在一个被引用列被修改（更新）的情况，可
选的动作相同。在这种情况下，CASCADE意味着被引用列的更新值应该被复制到引用行
中。

## 修改表

```sql
ALTER TABLE products ADD COLUMN description text;
ALTER TABLE products ADD COLUMN description text CHECK (description <> '');
ALTER TABLE products DROP COLUMN description;
#列中的数据将会消失。涉及到该列的表约束也会被移除。然而，如果该列被另一个表的外键
#所引用，PostgreSQL不会安静地移除该约束。我们可以通过增加CASCADE来授权移除任何
#依赖于被删除列的所有东西：
ALTER TABLE products DROP COLUMN description CASCADE;

ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id) REFERENCES
 product_groups;
# 要增加一个不能写成表约束的非空约束，可使用语法：
ALTER TABLE products ALTER COLUMN product_no SET NOT NULL;
ALTER TABLE products DROP CONSTRAINT some_name;
ALTER TABLE products ALTER COLUMN product_no DROP NOT NULL;

ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
ALTER TABLE products ALTER COLUMN price DROP DEFAULT;

ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2);

ALTER TABLE products RENAME COLUMN product_no TO product_number;
ALTER TABLE products RENAME TO items;


```

## 权限

有多种不同的权
限：SELECT、INSERT、UPDATE、DELETE、TRUNCATE、REFERENCES、TRIGGER、CREATE、CONNECT及USAGE。可以应用于一个特定对象的权限随着对象的类型（表、函数等）而不
同。PostgreSQL所支持的不同类型的完整权限信息请参考GRANT。下面的章节将简单介绍
如何使用这些权限。

要分配权限，可以使用GRANT命令。例如，如果joe是一个已有角色，而accounts是一个已有
表，更新该表的权限可以按如下方式授权：

```
GRANT UPDATE ON accounts TO joe;
```

用ALL取代特定权限会把与对象类型相关的所有权限全部授权。

为了撤销一个权限，使用REVOKE命令：

```
REVOKE ALL ON accounts FROM PUBLIC;
```

## 行安全策略

当在一个表上启用行安全性时（使用 ALTER TABLE ... ENABLE ROW LEVEL SECURITY），所有对该表选择行或者修改行的普通访问都必须被一条 行安全性策略所允许（不过，表的拥有者通常不服从行安全性策略）。如果 表上不存在策略，将使用一条默认的否定策略，即所有的行都不可见或者不能 被修改。应用在整个表上的操作不服从行安全性，例如TRUNCATE和 REFERENCES。

作为一个简单的例子，这里是如何在account关系上 创建一条策略以允许只有managers角色的成员能访问行， 并且只能访问它们账户的行：

```sql
CREATE TABLE accounts (manager text, company text, contact_email text);
ALTER TABLE accounts ENABLE ROW LEVEL SECURITY;
CREATE POLICY account_managers ON accounts TO managers
USING (manager = current_user);

```

如果没有指定角色或者使用了特殊的用户名PUBLIC， 则该策略适用于系统上所有的用户。
要允许所有用户访问users 表中属于他们自己的行，可以使用一条简单的策略：

```sql
CREATE POLICY user_policy ON users
USING (user_name = current_user);
```

要对相对于可见行是被增加到表中的行使用一条不同的策略，可以使用 WITH CHECK子句。这条策略将允许所有用户查看 users表中的所有行，但是只能修改它们自己的行：

```sql
CREATE POLICY user_policy ON users
USING (true)
WITH CHECK (user_name = current_user);
```

## 模式

```sql
CREATE SCHEMA myschema;
CREATE TABLE myschema.mytable (
...
);
DROP SCHEMA myschema;
# 要删除一个模式以及其中包含的所有对象，可用：
DROP SCHEMA myschema CASCADE;

#我们常常希望创建一个由其他人所拥有的模式（因为这是将用户动作限制在良定义的名字空
间中的方法之一）。其语法是：
CREATE SCHEMA schema_name AUTHORIZATION user_name;
```

​	在前面的小节中，我们创建的表都没有指定任何模式名称。默认情况下这些表（以及其他对象）会自动的被放入一个名为“public”的模式中。任何新数据库都包含这样一个模式。因此，下面的命令是等效的：

## 模式搜索路径

要显示当前搜索路径，使用下面的命令：
SHOW search_path;
在默认设置下这将返回：

```sql
search_path

"$user",public
```

第一个元素说明一个和当前用户同名的模式会被搜索。如果不存在这个模式，该项将被忽略。第二个元素指向我们已经见过的公共模式。

```sql
#要把新模式放在搜索路径中，我们可以使用：
SET search_path TO myschema,public;
#（我们在这里省略了$user，因为我们并不立即需要它）。然后我们可以该表而
#无需使用模式限定：
DROP TABLE mytable;
```

## 模式和权限

默认情况下，用户不能访问不属于他们的模式中的任何对象。要允许这种行为，模式的拥有者必须在该模式上授予USAGE权限。为了允许用户使用模式中的对象，可能还需要根据对象授予额外的权限。

## 系统目录模式

​	除public和用户创建的模式之外，每一个数据库还包括一个pg_catalog模式，它包含了系统表和所有内建的数据类型、函数以及操作符。pg_catalog总是搜索路径的一个有效部分。如果没有在路径中显式地包括该模式，它将在路径中的模式之前被搜索。这保证了内建的名称总是能被找到。然而，如果我们希望用用户定义的名称重载内建的名称，可以显式的将pg_catalog放在搜索路径的末尾。

## 表分区

PostgreSQL为以下形式的分区提供了内置支持：
**范围分区**
	该表被分区到由键列或列集定义的“范围”中， 分配给不同分区的值范围之间没有重叠。例如，可以按日期范围进行分区， 也可以按特定业务对象的标识符范围进行分区。
**列表分区**
	表通过明确列出每个分区中出现的键值进行分区。



1.通过声明PARTITION BY子句将 measurement表创建为分区表， 它包括分区方法（该例中是RANGE） 和要用作分区键的字段。

```sql
CREATE TABLE measurement (
city_id int not null,
logdate date not null,
peaktemp int,
unitsales int
) PARTITION BY RANGE (logdate);
```

2.创建分区

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
FOR VALUES FROM ('2006-02-01') TO ('2006-03-01')
CREATE TABLE measurement_y2006m03 PARTITION OF measurement
FOR VALUES FROM ('2006-03-01') TO ('2006-04-01')

...
CREATE TABLE measurement_y2007m11 PARTITION OF measurement
FOR VALUES FROM ('2007-11-01') TO ('2007-12-01')
CREATE TABLE measurement_y2007m12 PARTITION OF measurement
FOR VALUES FROM ('2007-12-01') TO ('2008-01-01')
TABLESPACE fasttablespace;

CREATE TABLE measurement_y2008m01 PARTITION OF measurement
FOR VALUES FROM ('2008-01-01') TO ('2008-02-01')
TABLESPACE fasttablespace
WITH (parallel_workers = 4);
```

要实现子分区，在创建单个分区的语句中声明PARTITION BY 子句，例如：

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
FOR VALUES FROM ('2006-02-01') TO ('2006-03-01')
PARTITION BY RANGE (peaktemp);
```

3. 对于每一个分区，在键列上创建索引，以及您可能需要的其他索引。

```sql
CREATE INDEX ON measurement_y2006m02 (logdate);
CREATE INDEX ON measurement_y2006m03 (logdate);
...
CREATE INDEX ON measurement_y2007m11 (logdate);
CREATE INDEX ON measurement_y2007m12 (logdate);
CREATE INDEX ON measurement_y2008m01 (logdate);
```

4. 确保在postgresql.conf中constraint_exclusion配置参数没有被禁用。如果它被禁用，查询将
不会被按照期望的方式优化。

在上面的例子中，我们将每个月创建一个新的分区， 所以编写一个脚本可以自动生成所需的
DDL。

# 数据操纵

## 从修改行中返回数据

```sql
UPDATE products SET price = price * 1.10
WHERE price <= 99.99
RETURNING name, price AS new_price;

DELETE FROM products
WHERE obsoletion_date = 'today'
RETURNING *;
```

# 查询

## 表表达式

### LATERAL子查询

​	可以在出现于FROM中的子查询前放置关键词LATERAL。这允许它们引用前面的FROM项提
供的列（如果没有LATERAL，每一个子查询将被独立计算，并且因此不能被其他FROM项
交叉引用）。

​	如果一个FROM项包含LATERAL交叉引用，计算过程如下：对于提供交叉引用列的FROM项
的每一行，或者多个提供这些列的多个FROM项的行集合，LATERAL项将被使用该行或者
行集中的列值进行计算。得到的结果行将和它们被计算出来的行进行正常的连接。对于来自
这些列的源表的每一行或行集，该过程将重复。

```sql
SELECT * FROM foo, LATERAL (SELECT * FROM bar WHERE bar.id = foo.bar_id) ss;
```

这不是非常有用，因为它和一种更简单的形式得到的结果完全一样：

```sql
SELECT * FROM foo, bar WHERE bar.id = foo.bar_id;
```

### GROUPING SETS、CUBE和ROLLUP

![](https://pic.imgdb.cn/item/6260c6bf239250f7c5682b0e.jpg)





![](https://pic.imgdb.cn/item/6260c6de239250f7c5687833.jpg)


![](https://pic.imgdb.cn/item/6260c6f4239250f7c568a7d4.jpg)



![](https://pic.imgdb.cn/item/6260c712239250f7c568e552.jpg)

## 组合查询

两个查询的结果可以用集合操作并、交、差进行组合。语法是

```sql
query1 UNION [ALL] query2
query1 INTERSECT [ALL] query2
query1 EXCEPT [ALL] query2
```



## LIMIT和OFFSET

```sql
SELECT select_list
FROM table_expression
[ ORDER BY ... ]
[ LIMIT { number | ALL } ] [ OFFSET number ]
```

## VALUES列表

VALUES提供了一种生成“常量表”的方法，它可以被使用在一个查询中而不需要实际在磁
盘上创建一个表。语法是：

```sql
VALUES ( expression [, ...] ) [, ...]
```

每一个被圆括号包围的表达式列表生成表中的一行。列表都必须具有相同数据的元素（即表
中列的数目），并且在每个列表中对应的项必须具有可兼容的数据类型。分配给结果的每一
列的实际数据类型使用和UNION相同的规则确定（参见第 10.5 节）。
一个例子：

```sql
VALUES (1, 'one'), (2, 'two'), (3, 'three');
```

将会返回一个有两列三行的表。它实际上等效于：

```sql
SELECT 1 AS column1, 'one' AS column2
UNION ALL
SELECT 2, 'two'
UNION ALL
SELECT 3, 'three';
```

## WITH查询（公共表表达式）

### WITH中的SELECT

```sql
WITH regional_sales AS (
SELECT region, SUM(amount) AS total_sales
FROM orders
GROUP BY region
), top_regions AS (
SELECT region
FROM regional_sales
WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
product,
SUM(quantity) AS product_units,
SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

## 递归查询

```sql
# 想下递归
WITH RECURSIVE r AS (
    SELECT * FROM public.crm_sys_menu WHERE id = v_pid
    union ALL
    SELECT public.crm_sys_menu.* FROM public.crm_sys_menu, r WHERE public.crm_sys_menu.p_id = r.id
)
select * from r;
```



1. 计算非递归项。对UNION（但不对UNION ALL），抛弃重复行。把所有剩余的行包括
   在递归查询的结果中，并且也把它们放在一个临时的工作表中。

2. 只要工作表不为空，重复下列步骤：
a. 计算递归项，用当前工作表的内容替换递归自引用。对UNION（不是UNION
ALL），抛弃重复行以及那些与之前结果行重复的行。将剩下的所有行包括在递归
查询的结果中，并且也把它们放在一个临时的中间表中。
b. 用中间表的内容替换工作表的内容，然后清空中间表。

## WITH中的数据修改语句

你可以在WITH中使用数据修改语句（INSERT、UPDATE或DELETE）。这允许你在同一个
查询中执行多个而不同操作。一个例子：

```sqk
WITH moved_rows AS (
DELETE FROM products
WHERE
"date" >= '2010-10-01' AND
"date" < '2010-11-01'
RETURNING *
)
INSERT INTO products_log
SELECT * FROM moved_rows;
```

​	这个查询实际上从products把行移动到products_log。WITH中的DELETE删除来自products的指定行，以它的RETURNING子句返回它们的内容，并且接着主查询读该输出并将它插入到products_log。

一个非特殊使用的例子：

```sql
WITH t AS (
DELETE FROM foo
)
DELETE FROM bar;
```

这个例子将从表foo和bar中移除所有行。被报告给客户端的受影响行的数目可能只包括从bar中移除的行。

# 数据类型

![](https://pic.imgdb.cn/item/6260ef8e239250f7c5b851b6.jpg)

![](https://pic.imgdb.cn/item/6260efa9239250f7c5b891e3.jpg)

![](https://pic.imgdb.cn/item/6260efbe239250f7c5b8c750.jpg)

## 数字类型

![](https://pic.imgdb.cn/item/6260efde239250f7c5b911f2.jpg)

![](https://pic.imgdb.cn/item/6260efe8239250f7c5b92b0d.jpg)

### 序列整形

```sql
CREATE TABLE tablename (
colname SERIAL
);
# 等价于以下语句：
CREATE SEQUENCE tablename_colname_seq;
CREATE TABLE tablename (
colname integer NOT NULL DEFAULT nextval('tablename_colname_seq')
);
ALTER SEQUENCE tablename_colname_seq OWNED BY tablename.colname;
```

> **注意**
>
> 因为smallserial、serial和bigserial是用序列实现的，所以即使没有删除过行，在出现在列中的序列值可能有“空洞”或者间隙。如果一个从序列中分配的值被用在一行中，即使该行最终没有被成功地插入到表中，该值也被“用掉”了。例如，当插入事务回滚时就会发生这种情况。更多信息参见第 9.16 节中的nextval()。

## 货币类型

![](https://pic.imgdb.cn/item/6260f06d239250f7c5ba3003.jpg)

## 字符类型

![](https://pic.imgdb.cn/item/6260f088239250f7c5ba6541.jpg)

![](https://pic.imgdb.cn/item/6260f0d4239250f7c5bafe79.jpg)

## 二进制数据类型

![](https://pic.imgdb.cn/item/62616555239250f7c5cbbe70.jpg)

### bytea的逃逸格式

![](https://pic.imgdb.cn/item/626165a0239250f7c5ccb1cd.jpg)

## 日期/时间类型

![](https://pic.imgdb.cn/item/626165ca239250f7c5cd2ed8.jpg)

​	SQL要求只写timestamp等效于timestamp without time zone，并且PostgreSQL鼓励这种行为。timestamptz被接受为timestamp with time zone的一种简写，这是一种PostgreSQL的扩展。

​	time、timestamp和interval接受一个可选的精度值 p，这个精度值声明在秒域中小数点之后保留的位数。缺省情况下，在精度上没有明确的边界，p允许的范围是从 0 到6。

​	interval类型有一个附加选项，它可以通过写下面之一的短语来限制存储的fields的集合：
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND



类型time with time zone是 SQL 标准定义的，但是该定义显示出了一些会影响可用性的性质。在大多数情况下， date、time、timestamp without time zone和timestamp with time zone的组合就应该能提供任何应用所需的全范围的日期/时间功能。

### 日期

![](https://pic.imgdb.cn/item/6261667d239250f7c5cede78.jpg)

### 时间

![](https://pic.imgdb.cn/item/626166cd239250f7c5cf9122.jpg)

![](https://pic.imgdb.cn/item/626166e3239250f7c5cfc64d.jpg)

### 时间戳

![](https://pic.imgdb.cn/item/62616701239250f7c5d00ff2.jpg)

### 特殊值

![](https://pic.imgdb.cn/item/6261672a239250f7c5d07547.jpg)

### 日期/时间输出

![](https://pic.imgdb.cn/item/6261674b239250f7c5d0c2ce.jpg)

### 间隔输入

![](https://pic.imgdb.cn/item/6261678e239250f7c5d16cdc.jpg)

## 布尔类型

![](https://pic.imgdb.cn/item/626167bb239250f7c5d1ea8e.jpg)

## 枚举类型

### 枚举类型的声明

枚举类型可以使用CREATE TYPE命令创建，例如：

```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
```

一旦被创建，枚举类型可以像很多其他类型一样在表和函数定义中使用：

```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TABLE person (
name text,
current_mood mood
);
INSERT INTO person VALUES ('Moe', 'happy');
SELECT * FROM person WHERE current_mood = 'happy';
name | current_mood
------+--------------
Moe | happy
(1 row)
```

### 排序

一个枚举类型的值的排序是该类型被创建时所列出的值的顺序。枚举类型的所有标准的比较操作符以及相关聚集函数都被支持。例如：

```sql
INSERT INTO person VALUES ('Larry', 'sad');
INSERT INTO person VALUES ('Curly', 'ok');
SELECT * FROM person WHERE current_mood > 'sad';
name | current_mood
-------+--------------
Moe | happy
Curly | ok
(2 rows)
SELECT * FROM person WHERE current_mood > 'sad' ORDER BY current_mood;
name | current_mood
-------+--------------
Curly | ok
Moe | happy
(2 rows)
SELECT name
FROM person
WHERE current_mood = (SELECT MIN(current_mood) FROM person);
name
-------
Larry
(1 row)
```

### 类型安全性

每一种枚举数据类型都是独立的并且不能和其他枚举类型相比较。看这样一个例子：

```sql
CREATE TYPE happiness AS ENUM ('happy', 'very happy', 'ecstatic');
CREATE TABLE holidays (
num_weeks integer,
happiness happiness
);
INSERT INTO holidays(num_weeks,happiness) VALUES (4, 'happy');
INSERT INTO holidays(num_weeks,happiness) VALUES (6, 'very happy');
INSERT INTO holidays(num_weeks,happiness) VALUES (8, 'ecstatic');
INSERT INTO holidays(num_weeks,happiness) VALUES (2, 'sad');
ERROR: invalid input value for enum happiness: "sad"
SELECT person.name, holidays.num_weeks FROM person, holidays
```

## 几何类型

![](https://pic.imgdb.cn/item/6261683e239250f7c5d338a3.jpg)

## 网络地址类型

![](https://pic.imgdb.cn/item/6261685c239250f7c5d37fae.jpg)

## 位串类型

位串就是一串 1 和 0 的串。它们可以用于存储和可视化位掩码。我们有两种类型的 SQL 位类型：bit(n)和bit varying(n)，其中 n是一个正整数。

```sql
CREATE TABLE test (a BIT(3), b BIT VARYING(5));
INSERT INTO test VALUES (B'101', B'00');
INSERT INTO test VALUES (B'10', B'101');
ERROR: bit string length 2 does not match type bit(3)
INSERT INTO test VALUES (B'10'::bit(3), B'101');
SELECT * FROM test;
```

## 文本搜索类型

​	PostgreSQL提供两种数据类型，它们被设计用来支持全文搜索，全文搜索是一种在自然语言的文档集合中搜索以定位那些最匹配一个查询的文档的活动。tsvector类型表示一个为文本搜索优化的形式下的文档，tsquery类型表示一个文本查询。第 12 章提供了对于这种功能的详细解释，并且第 9.13 节总结了相关的函数和操作符。

### tsvector

​	一个tsvector值是一个排序的可区分词位的列表，词位是被正规化合并了同一个词的不同变种的词（详见第 12 章）。排序和去重是在输入期间自动完成的，如下例所示：

```sql
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector;
tsvector
----------------------------------------------------
'a' 'and' 'ate' 'cat' 'fat' 'mat' 'on' 'rat' 'sat'
```

```sql
# 要表示包含空白或标点的词位，将它们用引号包围：
SELECT $$the lexeme ' ' contains spaces$$::tsvector;
tsvector
-------------------------------------------
' ' 'contains' 'lexeme' 'spaces' 'the'
```

### tsquery

一个tsquery值存储要用于搜索的词位，并且使用布尔操作符&（AND）、|（OR）和!（NOT）来组合它们，还有短语搜索操作符`<->`（FOLLOWED BY）。也有一种 FOLLOWED BY 操作符的变体`<N>`，其中N是一个整数常量，它指定要搜索的两个词位之间的距离。`<->`等效于`<1>`。

```sql
SELECT 'fat & rat'::tsquery;
tsquery
---------------
'fat' & 'rat'
SELECT 'fat & (rat | cat)'::tsquery;
tsquery
---------------------------
'fat' & ( 'rat' | 'cat' )
SELECT 'fat & rat & ! cat'::tsquery;
tsquery
------------------------
'fat' & 'rat' & !'cat'
```

## UUID类型

![](https://pic.imgdb.cn/item/62616969239250f7c5d5f535.jpg)

## XML类型

​	xml数据类型可以被用来存储XML数据。它比直接在一个text域中存储XML数据的优势在于，它会检查输入值的结构是不是良好，并且有支持函数用于在其上执行类型安全的操作，参见第 9.14 节。使用这种数据类型要求在安装时用configure --with-libxml选项编译。

### 创建XML值

![](https://pic.imgdb.cn/item/6261699b239250f7c5d6662f.jpg)

## JSON 类型

​	有两种 JSON 数据类型：json 和 jsonb。它们 几乎接受完全相同的值集合作为输入。主要的实际区别之一是 效率。json数据类型存储输入文本的精准拷贝，处理函数必须在每 次执行时必须重新解析该数据。而jsonb数据被存储在一种分解好的 二进制格式中，它在输入时要稍慢一些，因为需要做附加的转换。但是 jsonb在处理时要快很多，因为不需要解析。jsonb也支持索引，这也是一个令人瞩目的优势。

​	通常，除非有特别特殊的需要（例如遗留的对象键顺序假设），大多数应用应该 更愿意把JSON 数据存储为jsonb。

​	PostgreSQL对每个数据库只允许一种 字符集编码。因此 JSON 类型不可能严格遵守 JSON 规范，除非数据库编码 是 UTF8。尝试直接包括数据库编码中无法表示的字符将会失败。反过来，能在数据库编码中表示但是不在 UTF8 中的字符是被允许的。

![](https://pic.imgdb.cn/item/62616a11239250f7c5d777e4.jpg)

### jsonb 索引

GIN 索引可以被用来有效地搜索在大量jsonb文档（数据）中出现 的键或者键值对。提供了两种 GIN “操作符类”，它们在性能和灵活 性方面做出了不同的平衡。

![](https://pic.imgdb.cn/item/62616a66239250f7c5d85e10.jpg)

![](https://pic.imgdb.cn/item/62616a89239250f7c5d8b69b.jpg)

![](https://pic.imgdb.cn/item/62616af7239250f7c5d9cbc6.jpg)

## 数组

```sql
CREATE TABLE sal_emp (
name text,
pay_by_quarter integer[],
schedule text[][]
);
# CREATE TABLE的语法允许指定数组的确切大小，例如：
CREATE TABLE tictactoe (
squares integer[3][3]
);

```

​	然而，当前的实现忽略任何提供的数组尺寸限制，即其行为与未指定长度的数组相同。

​	当前的实现也不会强制所声明的维度数。一个特定元素类型的数组全部被当作是相同的类型，而不论其尺寸或维度数。因此，在CREATE TABLE中声明数组的尺寸或维度数仅仅只是文档而已，它并不影响运行时的行为。

另一种符合SQL标准的语法是使用关键词ARRAY，可以用来定义一维数组。pay_by_quarter可以这样定义：

```
pay_by_quarter integer ARRAY[4],
```

或者，不指定数组尺寸：

```sql
pay_by_quarter integer ARRAY,
```

但是和前面一样，PostgreSQL在任何情况下都不会强制尺寸限制。

### 数组值输入

```sql
'{ val1 delim val2 delim ... }'
'{{1,2,3},{4,5,6},{7,8,9}}'

INSERT INTO sal_emp
VALUES ('Bill',
'{10000, 10000, 10000, 10000}',
'{{"meeting", "lunch"}, {"training", "presentation"}}');
```

### 访问数组

现在，我们可以在该表上运行一些查询。首先，我们展示如何访问一个数组中的一个元素。下面的查询检索在第二季度工资发生变化的雇员的名字：

```sql
SELECT name FROM sal_emp WHERE pay_by_quarter[1] <> pay_by_quarter[2];

SELECT schedule[1:2][2] FROM sal_emp WHERE name = 'Bill';
```

`如果数组本身为空或者任何一个下标表达式为空，访问数组下标表达式将会返回空值。如果下标超过了数组边界，下标表达式也会返回空值（这种情况不会抛出错误）。例如，如果schedule目前具有的维度是

```
[1:3][1:2]
```

，那么引用`schedule[3][3]`将得到NULL。相似地，使用错误的下标号引用一个数组会得到空值而不是错误。

任何数组值的当前维度可以使用array_dims函数获得：

```sql
SELECT array_dims(schedule) FROM sal_emp WHERE name = 'Carol';
array_dims
------------
[1:2][1:2]
(1 row)
```

array_length将返回一个指定数组维度的长度：

```sql
SELECT array_length(schedule, 1) FROM sal_emp WHERE name = 'Carol';
```

cardinality返回一个数组中在所有维度上的元素总数。 这实际上是调用unnest将会得到的行数：

```sql
SELECT cardinality(schedule) FROM sal_emp WHERE name = 'Carol';
cardinality
-------------
4
(1 row)
```

### 修改数组

```sql
# 一个数组值可以被整个替换：
UPDATE sal_emp SET pay_by_quarter = '{25000,25000,27000,27000}'
WHERE name = 'Carol';
# 或者使用ARRAY表达式语法：
UPDATE sal_emp SET pay_by_quarter = ARRAY[25000,25000,27000,27000]
WHERE name = 'Carol';
# 一个数组也可以在一个元素上被更新：
UPDATE sal_emp SET pay_by_quarter[4] = 15000
WHERE name = 'Bill';
# 或者在一个切片上被更新：
UPDATE sal_emp SET pay_by_quarter[1:2] = '{27000,27000}'
WHERE name = 'Carol';
# 也可以使用省略lower-bound或者 upper-bound的切片语法，但是只能用于 更新一个不是NULL 或者零维的数组值（否则无法替换现有的下标界线）。

# 新的数组值也可以通过串接操作符||构建：
SELECT ARRAY[1,2] || ARRAY[3,4];
?column?
-----------
{1,2,3,4}
(1 row)

SELECT ARRAY[5,6] || ARRAY[[1,2],[3,4]];
?column?
---------------------
{{5,6},{1,2},{3,4}}
(1 row)
```

串接操作符允许把一个单独的元素加入到一个一维数组的开头或末尾。它也能接受两个N维数组，或者一个N维数组和一个N+1维数组。

```sql
# 当一个单独的元素被加入到一个一维数组的开头或末尾时，其结果是一个和数组操作数具有相同下界下标的新数组。例如：
SELECT array_dims(1 || '[0:1]={2,3}'::int[]);
array_dims
------------
[0:2]
(1 row)
SELECT array_dims(ARRAY[1,2] || 3);
array_dims
------------
[1:3]
(1 row)
# 当两个具有相同维度数的数组被串接时，其结果保留左操作数的外维度的下界下标。结果将是一个数组，它由左操作数的每一个元素以及紧接着的右操作数的每一个元素。例如：
SELECT array_dims(ARRAY[1,2] || ARRAY[3,4,5]);
array_dims
------------
[1:5]
(1 row)
SELECT array_dims(ARRAY[[1,2],[3,4]] || ARRAY[[5,6],[7,8],[9,0]]);
array_dims
------------
[1:5][1:2]
(1 row)
当一个N维数组被放在另一个N+1维数组的前面或者后面时，结果和上面的例子相似。每一
个N维子数组实际上是N+1维数组外维度的一个元素。例如：
SELECT array_dims(ARRAY[1,2] || ARRAY[[3,4],[5,6]]);
array_dims
------------
[1:3][1:2]
(1 row)
一个数组也可以通过使用函数array_prepend、array_append或array_cat构建。前两个函数仅支
持一维数组，但array_cat支持多维数组。 一些例子：
SELECT array_prepend(1, ARRAY[2,3]);
array_prepend
---------------
{1,2,3}
(1 row)
SELECT array_append(ARRAY[1,2], 3);
array_append
--------------
{1,2,3}
(1 row)
SELECT array_cat(ARRAY[1,2], ARRAY[3,4]);
array_cat
-----------
{1,2,3,4}
(1 row)
SELECT array_cat(ARRAY[[1,2],[3,4]], ARRAY[5,6]);
array_cat
---------------------
{{1,2},{3,4},{5,6}}
(1 row)
SELECT array_cat(ARRAY[5,6], ARRAY[[1,2],[3,4]]);
array_cat
---------------------
{{5,6},{1,2},{3,4}}
```

```sql
SELECT ARRAY[1, 2] || '{3, 4}'; -- 没有指定类型的文字被当做一个数组
?column?
-----------
{1,2,3,4}
SELECT ARRAY[1, 2] || '7'; -- 这个也是
ERROR: malformed array literal: "7"
SELECT ARRAY[1, 2] || NULL; -- 未修饰的 NULL 也是如此
?column?
----------
{1,2}
(1 row)
SELECT array_append(ARRAY[1, 2], NULL); -- 这可能才是想要的意思
array_append
--------------
{1,2,NULL}
```

### 在数组中搜索

```sql
SELECT * FROM sal_emp WHERE pay_by_quarter[1] = 10000 OR
pay_by_quarter[2] = 10000 OR
pay_by_quarter[3] = 10000 OR
pay_by_quarter[4] = 10000;

SELECT * FROM sal_emp WHERE 10000 = ANY (pay_by_quarter);

# 此外，我们还可以查找所有元素值都为10000的数组所在的行：
SELECT * FROM sal_emp WHERE 10000 = ALL (pay_by_quarter);
# 另外，generate_subscripts函数也可以用来完成类似的查找。例如：
SELECT * FROM
(SELECT pay_by_quarter,
generate_subscripts(pay_by_quarter, 1) AS s
FROM sal_emp) AS foo
WHERE pay_by_quarter[s] = 10000;
```

我们也可以使用&&操作符来搜索一个数组，它会检查左操作数是否与右操作数重叠。例如：

```sql
SELECT * FROM sal_emp WHERE pay_by_quarter && ARRAY[10000];
```

​	你也可以使用array_position和array_positions在一个 数组中搜索特定值。前者返回值在数组中第一次出现的位置的下标。后者返回一个数组， 其中有该值在数组中的所有出现位置的下标。例如：

```sql
SELECT array_position(ARRAY['sun','mon','tue','wed','thu','fri','sat'], 'mon');
array_positions
-----------------
2
SELECT array_positions(ARRAY[1, 4, 3, 1, 3, 4, 2, 1], 1);
array_positions
-----------------
{1,4,8}
```

> 数组不是集合，在其中搜索指定数组元素可能是数据设计失误的表现。考虑使用一个独立的表来替代，其中每一行都对应于一个数组元素。这将更有利于搜索，并且对于大量元素的可扩展性更好。

## 复合类型

​	一个复合类型表示一行或一个记录的结构，它本质上就是一个域名和它们数据类型的列表。PostgreSQL允许把复合类型用在很多能用简单类型的地方。例如，一个表的一列可以被声明为一种复合类型。

### 复合类型的声明

```sql
CREATE TYPE complex AS (
r double precision,
i double precision
);
CREATE TYPE inventory_item AS (
name text,
supplier_id integer,
price numeric
);
```

该语法堪比CREATE TABLE，不过只能指定域名和类型，当前不能包括约束（例如NOT NULL）。注意AS关键词是必不可少的，如果没有它，系统将认为用户想要的是一种不同类 型的CREATE TYPE命令，并且你将得到奇怪的语法错误。

定义了类型之后，我们可以用它们来创建表：

```sql
CREATE TABLE on_hand (
item inventory_item,
count integer
);

INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);
# or functions:
CREATE FUNCTION price_extension(inventory_item, integer) RETURNS numeric
AS 'SELECT $1.price * $2' LANGUAGE SQL;
```

### 构造组合值

```sql
'( val1 , val2 , ... )'
'("fuzzy dice",42,1.99)'
```

这将是上文定义的inventory_item类型的一个合法值。要让一个域为 NULL，在列表中它的位置上根本不写字符。例如，这个常量指定其第三个域为 NULL：

```sql
'("fuzzy dice",42,)'
```

如果你写一个空字符串而不是 NULL，写上两个引号：

```sql
'("",42,)'  
```

### 访问复合类型

```sql
SELECT item.name FROM on_hand WHERE item.price > 9.99;
SELECT (item).name FROM on_hand WHERE (item).price > 9.99;
SELECT (on_hand.item).name FROM on_hand WHERE (on_hand.item).price > 9.99;
```

### 修改复合类型

```sql
INSERT INTO mytab (complex_col) VALUES((1.1,2.2));
UPDATE mytab SET complex_col = ROW(1.1,2.2) WHERE ...;
UPDATE mytab SET complex_col.r = (complex_col).r + 1 WHERE ...;
INSERT INTO mytab (complex_col.r, complex_col.i) VALUES(1.1, 2.2);
```

### 在查询中使用复合类型

​	在PostgreSQL中，在查询中引用表名（或别名） 是对该表的当前行的复合类型的有效引用。例如，如果我们有下面这样一个表 inventory_item：

```sql
SELECT c FROM inventory_item c;
```

​	不过请注意，简单名称首先匹配列名然后再匹配表名， 所以这个示例能有效是因为查询表中没有列的名字为c。

## 范围类型

​	范围类型是表达某种元素类型（称为范围的subtype）的一个值的范围的数据类型。例如，timestamp的范围可以被用来表达一个会议室被保留的时间范围。在这种情况下，数据类型是tsrange（“timestamp range”的简写）而timestamp是 subtype。subtype 必须具有一种总体的顺序，这样对于元素值是在一个范围值之内、之前或之后就是界线清楚的。

### 内建范围类型

PostgreSQL 带有下列内建范围类型：
• int4range — integer的范围
• int8range — bigint的范围
• numrange — numeric的范围
• tsrange — 不带时区的 timestamp的范围
• tstzrange — 带时区的 timestamp的范围
• daterange — date的范围
此外，你可以定义自己的范围类型，详见CREATE TYPE。

### 例子

```sql
CREATE TABLE reservation (room int, during tsrange);
INSERT INTO reservation VALUES
(1108, '[2010-01-01 14:30, 2010-01-01 15:30)');
-- 包含
SELECT int4range(10, 20) @> 3;
-- 重叠
SELECT numrange(11.1, 22.2) && numrange(20.0, 30.0);
-- 抽取上界
SELECT upper(int8range(15, 25));
-- 计算交集
SELECT int4range(10, 20) * int4range(15, 25);
-- 范围为空吗？
SELECT isempty(numrange(1, 5));
```

### 包含和排除边界

​	每一个非空范围都有两个界限，下界和上界。这些值之间的所有点都被包括在范围内。一个包含界限意味着边界点本身也被包括在范围内，而一个排除边界意味着边界点不被包括在范围内。

### 无限（无界）范围

​	一个范围的下界可以被忽略，意味着所有小于上界的点都被包括在范围中。同样，如果范围的上界被忽略，那么所有比上界大的的都被包括在范围中。如果上下界都被忽略，该元素类型的所有值都被认为在该范围中。

### 范围输入/输出

一个范围值的输入必须遵循下列模式之一：

```sql
(lower-bound,upper-bound)
(lower-bound,upper-bound]
[lower-bound,upper-bound)
[lower-bound,upper-bound]
empty
 
 -- 包括 3，不包括 7，并且包括 3 和 7 之间的所有点
SELECT '[3,7)'::int4range;
-- 既不包括 3 也不包括 7，但是包括之间的所有点
SELECT '(3,7)'::int4range;
-- 只包括单独一个点 4
SELECT '[4,4]'::int4range;
-- 不包括点（并且将被标准化为 '空'）
SELECT '[4,4)'::int4range;
```

### 构造范围

```sql
-- 完整形式是：下界、上界以及指示界限包含性/排除性的文本参数。
SELECT numrange(1.0, 14.0, '(]');
-- 如果第三个参数被忽略，则假定为 '[)'。
SELECT numrange(1.0, 14.0);
-- 尽管这里指定了 '(]'，显示时该值将被转换成标准形式，因为 int8range 是一种离散范围类型（见下文）。
SELECT int8range(1, 14, '(]');
-- 为一个界限使用 NULL 导致范围在那一边是无界的。
SELECT numrange(NULL, 2.2);
```

### 离散范围类型

​	一种范围的元素类型具有一个良定义的“步长”，例如integer或date。在这些类型中，如果两个元素之间没有合法值，它们可以被说成是相邻。这与连续范围相反，连续范围中总是（或者几乎总是）可以在两个给定值之间标识其他元素值。例如，numeric类型之上的一个范围就是连续的，timestamp上的范围也是（尽管timestamp具有有限的精度，并且在理论上可以被当做离散的，最好认为它是连续的，因为通常并不关心它的步长）。

### 定义新的范围类型

```sql
CREATE TYPE floatrange AS RANGE (
subtype = float8,
subtype_diff = float8mi
);
SELECT '[1.234, 5.678]'::floatrange;
```

因为float8没有有意义的“步长”，我们在这个例子中没有定义一个正规化函数。

### 索引

```sql
# 可以为范围类型的表列创建 GiST 和 SP-GiST 索引。例如，要创建一个 GiST 索引：
CREATE INDEX reservation_idx ON reservation USING GIST (during);
# 一个 GiST 或 SP-GiST 索引可以加速涉及以下范围操作符的查询： =、 &&、 <@、 @>、<<、 >>、 -|-、 &<以及 &> （详见表 9.50）。
```

### 范围上的约束

​	虽然UNIQUE是标量值的一种自然约束，它通常不适合于范围类型。反而，一种排除约束常常更加适合（见CREATE TABLE ... CONSTRAINT ... EXCLUDE）。排除约束允许在一个范围类型上说明诸如“non-overlapping”的约束。例如：

```sql
CREATE TABLE reservation (
during tsrange,
EXCLUDE USING GIST (during WITH &&)
);
```

该约束将阻止任何重叠值同时存在于表中：

## 对象标识符类型

​	对象标识符（OID）被PostgreSQL用来在内部作为多个系统表的主键。OID不会被添加到用户创建的表中，除非在创建表时指定了WITH OIDS或者default_with_oids配置变量被启用。类型oid表示一个对象标识符。也有多个oid的别名类型：regproc、regprocedure、regoper、regoperator、regclass、regtype、regrole、regnames8p.2ac4e显、regconfig和regdictionary示了一个概览。



​	OID的别名类型除了特定的输入和输出例程之外没有别的操作。这些例程可以接受并显示系统对象的符号名，而不是类型oid使用的原始数字值。别名类型使查找对象的OID值变得简单。例如，要检查与一个表mytable有关的pg_attribute行，你可以写：

```sql
SELECT * FROM pg_attribute WHERE attrelid = 'mytable'::regclass;
```

![](https://pic.imgdb.cn/item/6266b556239250f7c52da9ad.jpg)

## pg_lsn Type

​	pg_lsn数据类型可以被用来存储 LSN（日志序列号）数据，LSN 是一个指向 WAL 中的位置的指针。这个类型是XLogRecPtr的一种表达并且是 PostgreSQL的一种内部系统类型。

​	在内部，一个 LSN 是一个 64 位整数，表示在预写式日志流中的一个字节位置。它被打印成两个最高 8 位的十六进制数，中间用斜线分隔，例如16/B374D848。 pg_lsn类型支持标准的比较操作符，如=和 >。两个 LSN 可以用-操作符做减法， 结果将是分隔两个预写式日志位置的字节数。

## 伪类型

![](https://pic.imgdb.cn/item/62638908239250f7c5b1cdcf.jpg)

![](https://pic.imgdb.cn/item/62638915239250f7c5b1f844.jpg)

# 函数和操作符

![](https://pic.imgdb.cn/item/62668f41239250f7c5a4a192.jpg)

![](https://pic.imgdb.cn/item/62669026239250f7c5a6c0ea.jpg)

## 数学函数和操作符

![](https://pic.imgdb.cn/item/6266904f239250f7c5a71caa.jpg)

![](https://pic.imgdb.cn/item/62669060239250f7c5a7473e.jpg)

![](https://pic.imgdb.cn/item/6266906d239250f7c5a76606.jpg)

![](https://pic.imgdb.cn/item/6266907b239250f7c5a78b1c.jpg)



![](https://pic.imgdb.cn/item/62669091239250f7c5a7c568.jpg)

## 字符串函数和操作符

![](https://pic.imgdb.cn/item/626690ad239250f7c5a80666.jpg)

![](https://pic.imgdb.cn/item/626690c4239250f7c5a8419f.jpg)

![](https://pic.imgdb.cn/item/626690d3239250f7c5a8639f.jpg)

![](https://pic.imgdb.cn/item/626690e0239250f7c5a883f7.jpg)

![](https://pic.imgdb.cn/item/626690ef239250f7c5a8aa30.jpg)

![](https://pic.imgdb.cn/item/626690fc239250f7c5a8c5ba.jpg)

![](https://pic.imgdb.cn/item/6266910c239250f7c5a8eb2c.jpg)

![](https://pic.imgdb.cn/item/62669119239250f7c5a909a7.jpg)

![](https://pic.imgdb.cn/item/6266912f239250f7c5a93bf1.jpg)

## 模式匹配

![](https://pic.imgdb.cn/item/62669227239250f7c5ab806a.jpg)

一些例子：
```sql
'abc' LIKE 'abc' true
'abc' LIKE 'a%' true
'abc' LIKE '_b_' true
'abc' LIKE 'c' false
```

> **注意**
>
> 如果你关掉了standard_conforming_strings，你在文串常量中写的任何反斜线都需要被双写。详见第 4.1.2.1 节。

### SIMILAR TO正则表达式

```sql
string SIMILAR TO pattern [ESCAPE escape-character]
string NOT SIMILAR TO pattern [ESCAPE escape-character]
```

除了这些从LIKE借用的功能之外，SIMILAR TO支持下面这些从 POSIX 正则表达式借用的 模式匹配元字符：

* `|`表示选择（两个候选之一）。
* `*`表示重复前面的项零次或更多次。
*  `+`表示重复前面的项一次或更多次。
* `?`表示重复前面的项零次或一次。
* `{m}`表示重复前面的项刚好m次。
* `{m,}`表示重复前面的项m次或更多次。
* `{m,n}`表示重复前面的项至少m次并且不超过n次。
* 可以使用圆括号()把多个项组合成一个逻辑项。
*  一个方括号表达式[...]声明一个字符类，就像 POSIX 正则表达式一样。
  注意

![](https://pic.imgdb.cn/item/62669314239250f7c5ada95c.jpg)

![](https://pic.imgdb.cn/item/62669335239250f7c5adfd1d.jpg)

### POSIX正则表达式

![](https://pic.imgdb.cn/item/62669359239250f7c5ae5732.jpg)

```sql
substring('foobar' from 'o.b') oob
substring('foobar' from 'o(.)b') o
regexp_replace('foobarbaz', 'b..', 'X')
fooXbaz
regexp_replace('foobarbaz', 'b..', 'X', 'g')
fooXX
regexp_replace('foobarbaz', 'b(..)', E'X\\1Y', 'g')
fooXarYXazY

SELECT regexp_match('foobarbequebaz', 'bar.*que');
regexp_match
--------------
{barbeque}
(1 row)
SELECT regexp_match('foobarbequebaz', '(bar)(beque)');
regexp_match
--------------
{bar,beque}
(1 row)

# 在通常情况下，您只需要匹配的整个子字符串或NULL来匹配，请写下类似的内容
SELECT (regexp_match('foobarbequebaz', 'bar.*que'))[1];
regexp_match
--------------
barbeque
(1 row)


```

![](https://pic.imgdb.cn/item/626693f3239250f7c5afc53c.jpg)

![](https://pic.imgdb.cn/item/626693ff239250f7c5afe2c6.jpg)

## 数据类型格式化函数

![](https://pic.imgdb.cn/item/62669456239250f7c5b0a9dc.jpg)

![](https://pic.imgdb.cn/item/62669469239250f7c5b0d662.jpg)

![](https://pic.imgdb.cn/item/62669477239250f7c5b0f7cb.jpg)

## 时间/日期函数和操作符

![](https://pic.imgdb.cn/item/6266951d239250f7c5b282e1.jpg)



![](https://pic.imgdb.cn/item/62669562239250f7c5b31dd9.jpg)

![](https://pic.imgdb.cn/item/62669574239250f7c5b34802.jpg)

![](https://pic.imgdb.cn/item/62669582239250f7c5b36a2a.jpg)

### EXTRACT, date_part

![](https://pic.imgdb.cn/item/6266959d239250f7c5b3aaaf.jpg)

![](https://pic.imgdb.cn/item/626695ae239250f7c5b3d486.jpg)

![](https://pic.imgdb.cn/item/626695ba239250f7c5b3f269.jpg)

![](https://pic.imgdb.cn/item/626695c6239250f7c5b4120b.jpg)

![](https://pic.imgdb.cn/item/626695d9239250f7c5b4409b.jpg)

### date_trunc

![](https://pic.imgdb.cn/item/62669607239250f7c5b4a3e9.jpg)

### AT TIME ZONE

AT TIME ZONE结构允许把时间戳转换成不同的时区。

### 当前日期/时间

PostgreSQL提供了许多返回当前日期和时间的函数。这些 SQL 标准的函数全部都按照当前事务的开始时刻返回值：
```sql
CURRENT_DATE
CURRENT_TIME
CURRENT_TIMESTAMP
CURRENT_TIME(precision)
CURRENT_TIMESTAMP(precision)
LOCALTIME
LOCALTIMESTAMP
LOCALTIME(precision)
LOCALTIMESTAMP(precision)
```

CURRENT_TIME和CURRENT_TIMESTAMP传递带有时区的
值；LOCALTIME和LOCALTIMESTAMP传递的值不带时区

PostgreSQL同样也提供了返回当前语句开始时间的函数， 它们会返回函数被调用时的真实当前时间。这些非 SQL 标准的函数列表如下：

```sql
transaction_timestamp()
statement_timestamp()
clock_timestamp()
timeofday()
now()
```

### 延时执行

![](https://pic.imgdb.cn/item/6266966c239250f7c5b58968.jpg)

## 枚举支持函数

![](https://pic.imgdb.cn/item/62669696239250f7c5b5f99f.jpg)

## 文本搜索函数和操作符

略

## JSON 函数和操作符

![](https://pic.imgdb.cn/item/626696f4239250f7c5b6e8d4.jpg)

**表 9.44. 额外的jsonb操作符**

![](https://pic.imgdb.cn/item/6266972c239250f7c5b770d8.jpg)

**表 9.45. JSON 创建函数**

![](https://pic.imgdb.cn/item/62669781239250f7c5b84f5b.jpg)

![](https://pic.imgdb.cn/item/6266978f239250f7c5b876d0.jpg)

其他 略

## 序列操作函数

![](https://pic.imgdb.cn/item/626697b4239250f7c5b8e456.jpg)

## 条件表达式

### CASE

![](https://pic.imgdb.cn/item/626697d6239250f7c5b94012.jpg)

### COALESCE

```sql
COALESCE(value [, ...])
```

COALESCE函数返回它的第一个非空参数的值。当且仅当所有参数都为空时才会返回空。它常用于在为显示目的检索数据时用缺省值替换空值。

### NULLIF

```sql
NULLIF(value1, value2)
```

当value1和value2相等时，NULLIF返回一个空值。 否则它返回value1。 这些可以用于执行前文给出的COALESCE例子的逆操作：

### GREATEST和LEAST

![](https://pic.imgdb.cn/item/62669824239250f7c5b9eb5a.jpg)

### 聚集函数

![](https://pic.imgdb.cn/item/62669858239250f7c5ba8109.jpg)

![](https://pic.imgdb.cn/item/62669863239250f7c5ba9e69.jpg)

## 窗口函数

![](https://pic.imgdb.cn/item/626698b7239250f7c5bb7e6f.jpg)

![](https://pic.imgdb.cn/item/626698c3239250f7c5bb9d7f.jpg)

## 子查询表达式

* EXISTS
* IN
* NOT IN
  * 通常，表达式或者子查询行里的空值是按照 SQL 布尔表达式的一般规则进行组合
    的。 如果两个行对应的成员都非空并且相等，那么认为这两行相等；如果任意对应成员为非空且不等，那么这两行不等； 否则这样的行比较的结果是未知（空值）。如果所有行的结果要么是不等， 要么是空值，并且至少有一个空值，那么NOT IN的结果是空值。

### ANY/SOME

![](https://pic.imgdb.cn/item/6266991f239250f7c5bc82d0.jpg)

### ALL

![](https://pic.imgdb.cn/item/62669948239250f7c5bce287.jpg)

## 系统信息函数

![](https://pic.imgdb.cn/item/626699af239250f7c5bdfae7.jpg)

# 类型转换

略

# 索引

```sql
CREATE INDEX test1_id_index ON test1 (id);
```

索引的名字test1_id_index可以自由选择，但我们最好选择一个能让我们想起该索引用途的名字。

为了移除一个索引，可以使用DROP INDEX命令。索引可以随时被创建或删除。



一旦一个索引被创建，就不再需要进一步的干预：系统会在表更新时更新索引，而且会在它觉得使用索引比顺序扫描表效率更高时使用索引。但我们可能需要定期地运行ANALYZE命令来更新统计信息以便查询规划器能做出正确的决定。

## 索引类型

​	PostgreSQL提供了多种索引类型： B-tree、Hash、GiST、SP-GiST 、GIN 和 BRIN。

B-tree索引也可以用于检索排序数据。这并不会总是比简单扫描和排序更快，但是总是有用的。

Hash索引只能处理简单等值比较。不论何时当一个索引列涉及到一个使用了=操作符的比较时，查询规划器将考虑使用一个Hash索引。下面的命令将创建一个Hash索引：

```sql
CREATE INDEX name ON table USING HASH (column);
```

​	GiST索引并不是一种单独的索引，而是可以用于实现很多不同索引策略的基础设施。相应地，可以使用一个GiST索引的特定操作符根据索引策略（操作符类）而变化。作为一个例子，PostgreSQL的标准捐献包中包括了用于多种二维几何数据类型的GiST操作符类，它用来支持使用下列操作符的索引化查询。

## 多列索引

```sql
CREATE INDEX test2_mm_idx ON test2 (major, minor);
```

目前，只有 B-tree、GiST、GIN 和 BRIN 索引类型支持多列索引，最多可以指定32个列。

限制可以在源代码文件pg_config_manual.h中修改，但是修改后需要重新编译PostgreSQL。

## 索引和ORDER BY

​	除了简单地查找查询要返回的行外，一个索引可能还需要将它们以指定的顺序传递。这使得查询中的ORDER BY不需要独立的排序步骤。在PostgreSQL当前支持的索引类型中，只有Btree可以产生排序后的输出，其他索引类型会把行以一种没有指定的且与实现相关的顺序返回。

![](https://pic.imgdb.cn/item/62669b8f239250f7c5c2e6d8.jpg)

### 组合多个索引

​	只有查询子句中在索引列上使用了索引操作符类中的操作符并且通过AND连接时才能使用单一索引。例如，给定一个(a, b) 上的索引，查询条件WHERE a = 5 AND b = 6可以使用该索引，而查询WHERE a = 5 OR b = 6不能直接使用该索引。

## 唯一索引

索引也可以被用来强制列值的唯一性，或者是多个列组合值的唯一性。

```sql
CREATE UNIQUE INDEX name ON table (column [, ...]);
```

不需要手工在唯一列上创建索引，如果那样做也只是重复了自动创建的索引而
已。

### 表达式索引

```sql
SELECT * FROM test1 WHERE lower(col1) = 'value';
```

这种查询可以利用一个建立在lower(col1)函数结果之上的索引：

```sql
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));
```

## 部分索引

```sql
CREATE INDEX access_log_client_ip_ix ON access_log (client_ip)
WHERE NOT (client_ip > inet '192.168.100.0' AND
client_ip < inet '192.168.100.255');
```

## 操作符类和操作符族

一个索引定义可以为索引中的每一列都指定一个操作符类。
```sql
CREATE INDEX name ON table (column opclass [sort options] [, ...]);
```

操作符类text_pattern_ops、varchar_pattern_ops和 bpchar_pattern_ops分别支持类
型text、varchar和 char上的B树索引。它们与默认操作符类的区别是值的比较是严格按照字符进行而不是根据区域相关的排序规则。这使得这些操作符类适合于当一个数据库没有使用标准“C”区域时被使用在涉及模式匹配表达式（LIKE或POSIX正则表达式）的查询中。
一个例子是，你可以这样索引一个varchar列：

```sql
CREATE INDEX test_index ON test_table (col varchar_pattern_ops);
```

## 索引和排序规则

一个索引在每一个索引列上只能支持一种排序规则。如果需要多种排序规则，你可能需要多个索引。

## 只用索引的扫描

​	PostgreSQL中的所有索引都是二级索引，表示每一个索引都被存储在表的主数据区域（在PostgreSQL术语中被称为该表的堆）之外。这意味着在一次普通索引扫描中，每次取一行需要从索引和堆中取得数据。此外，虽然满足一个给定的可索引WHERE条件的索引项通常在索引中都靠拢在一起，但是它们所引用的表行可能分布在堆中的任何地方。因此一次索引扫描的堆访问部分可能会涉及到很多对堆的随机访问，这可能会很慢，尤其在传统的磁盘上（如第 11.5 节中所述，位图扫描试图通过有序地执行堆访问来减轻这种开销，但是效果也就那样而已）。

## 检查索引使用

* 总是先运行ANALYZE。这个命令会收集有关表中值分布情况的统计信息。估计一个查询将要返回的行数需要这些信息，而结果行数则被规划器用来为每一个可能的查询计划分配实际的代价。如果没有任何真实的统计信息，将会假定一些默认值，这几乎肯定是不准确的。在没有运行的情况下检查一个应用的索引使用情况是注定要失败的。详见第 24.1.3 节和第 24.1.6 节。
* 使用真实数据进行实验。使用测试数据来建立索引将会告诉你测试数据需要什么样的索引，但这并不代表真实数据的需要。



* 如果索引没有被用到，强制使用它们将会对测试非常有用。有一些运行时参数可以关闭多种计划类型（参见第 19.7.1 节）。例如，关闭顺序扫描（enable_seqscan）以及嵌套循环连接（enable_nestloop）将强制系统使用一种不同的计划。如果系统仍然选择使用一个顺序扫描或嵌套循环连接，则索引没有被使用的原因可能更加根本，例如查询条件不匹配索引（哪种查询能够使用哪种索引已经在前面的小节中解释过了）
* 如果强制索引使用确实使用了索引，则有两种可能性：系统是正确的并且索引确实不合适，或者查询计划的代价估计并没有反映真实情况。因此你应该对用索引的查询和不用索引的查询计时。此时EXPLAIN ANALYZE命令就能发挥作用了
* 如果发现代价估计是错误的，也分为两种可能性。总代价是用每个计划节点的每行代价乘以计划节点的选择度估计来计算的。计划节点的代价估计可以通过运行时参数调整（如第 19.7.2 节所述）。不准确的选择度估计可能是由于缺乏统计信息，可以通过调节统计信息收集参数（见ALTER TABLE）来改进。

# 全文搜索

## 介绍

​	文本搜索操作符已经在数据库中存在很多年了。PostgreSQL对文本数据类型提供了~、~*、LIKE和ILIKE操作符，但是它们缺少现代信息系统所要求的很多基本属性。

全文索引允许文档被预处理并且保存一个索引用于以后快速的搜索。预处理包括：

​	将文档解析成记号。标识出多种类型的记号是有所帮助的，例如数字、词、复杂的词、电子邮件地址，这样它们可以被以不同的方式处理。原则上记号分类取决于相关的应用，但是对于大部分目的都可以使用一套预定义的分类。PostgreSQL使用一个解析器来执行这个步骤。其中提供了一个标准的解析器，并且为特定的需要也可以创建定制的解析器。

​	将记号转换成词位。和一个记号一样，一个词位是一个字符串，但是它已经被正规化，这样同一个词的不同形式被变成一样。例如，正规化几乎总是包括将大写字母转换成小写形式，并且经常涉及移除后缀（例如英语中的s或es）。这允许搜索找到同一个词的变体形式，而不需要冗长地输入所有可能的变体。此外，这个步骤通常会消除停用词，它们是那些太普通的词，它们对于搜索是无用的（简而言之，记号是文档文本的原始片段，而词位是那些被认为对索引和搜索有用的词）。PostgreSQL使用词典来执行这个步骤。已经提供了多种标准词典，并且为特定的需要也可以创建定制的词典。

### 基本文本匹配

​	PostgreSQL中的全文搜索基于匹配操作符@@，它在一个tsvector（文档）匹配一个tsquery（查询）时返回true。哪种数据类型写在前面没有影响：

```sql
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;
?column?
----------
t
SELECT 'fat & cow'::tsquery @@ 'a fat cat sat on a mat and ate a fat rat'::tsvector;
?column?
----------
f
```

正如以上例子所建议的，一个tsquery并不只是一个未经处理的文本， 顶多一个tsvector是这样。一个tsquery包含搜索术语， 它们必须是已经正规化的词位，并且可以使用AND 、OR、NOT 以及 FOLLOWED BY 操作符结合多个术语 （语法细节请见第 8.11.2 节）。有几个函数to_tsquery、plainto_tsquery以及phraseto_tsquery可用于将用户书写的文本转换为正确的tsquery，它们会主要采用正则化出现在文本中的词的方法。相似地，to_tsvector被用来解析和正规化一个文档字符串。因此在实际上一个文本搜索匹配可能
看起来更像：

```sql
SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat');
?column?
----------
t
```

在tsquery中，&（AND）操作符指定它的两个参数都必须出现在文档中才表示匹配。类似地，|（OR）操作符指定至少一个参数必须出现，而!（NOT）操作符指定它的参数不出现才能匹配。 例如，查询fat & ! rat匹配包含fat但是不包含 rat的文档。



在<->（FOLLOWED BY） tsquery操作符的帮助下搜索可能的短语，只有该操作符的参数的匹配是相邻的并且符合给定顺序时，该操作符才算是匹配。例如：

```sql
SELECT to_tsvector('fatal error') @@ to_tsquery('fatal <-> error');
?column?
----------
t

SELECT to_tsvector('error is not fatal') @@ to_tsquery('fatal <-> error');
?column?
----------
f
```

FOLLOWED BY 操作符还有一种更一般的版本，形式是`<N>`，其中N是一个表示匹配词位位置之间的差。`<1>`和`<->`相同，而`<2>`允许刚好一个其他词位出现在匹配之间，以此类推。当有些词是停用词时，`phraseto_tsquery`函数利用这个操作符来构造一个能够匹配多词短语的tsquery。例如：

一种有时候有用的特殊情况是，`<0>`可以被用来要求两个匹配同一个词的模式。

## 表和索引

### 搜索一个表

```sql
SELECT title
FROM pgweb
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'friend');
```

以上的查询指定要使用english配置来解析和正规化字符串。我们也可以忽略配置参数：

```sql
SELECT title
FROM pgweb
WHERE to_tsvector(body) @@ to_tsquery('friend');
```

### 创建索引

```sql
CREATE INDEX pgweb_idx ON pgweb USING GIN(to_tsvector('english', body));
```

索引甚至可以连接列：

```sql
CREATE INDEX pgweb_idx ON pgweb USING GIN(to_tsvector('english', title || ' ' || body));
```

另一种方法是创建一个单独的tsvector列来保存to_tsvector的输出。这个例子是title和body的连接，使用coalesce来保证当其他域为NULL时一个域仍然能留在索引中：

```sql
ALTER TABLE pgweb ADD COLUMN textsearchable_index_col tsvector;
UPDATE pgweb SET textsearchable_index_col =
to_tsvector('english', coalesce(title,'') || ' ' || coalesce(body,''));
```

然后我们创建一个GIN索引来加速搜索：

```sql
CREATE INDEX textsearch_idx ON pgweb USING GIN(textsearchable_index_col);
```

## 空值文本搜索

​	要实现全文搜索必须要有一个从文档创建tsvector以及从用户查询创建tsquery的函数。而且我们需要一种有用的顺序返回结果，因此我们需要一个函数能够根据文档与查询的相关性比较文档。还有一点重要的是要能够很好地显示结果。PostgreSQL对所有这些函数都提供了支持。

### 解析文档

PostgreSQL提供了函数to_tsvector将一个文档转换成tsvector数据类型。

```sql
to_tsvector([ config regconfig, ] document text) returns tsvector
```

to_tsvector把一个文本文档解析成记号，把记号缩减成词位，并且返回一个tsvector，它列出
了词位以及词位在文档中的位置。文档被根据指定的或默认的文本搜索配置来处理。下面是
一个简单例子：

```sql
SELECT to_tsvector('english', 'a fat cat sat on a mat - it ate a fat rats');

to_tsvector
-----------------------------------------------------
'ate':9 'cat':3 'fat':2,11 'mat':7 'rat':12 'sat':4
```

### 解析查询

​	PostgreSQL提供了函数to_tsquery、plainto_tsquery和phraseto_tsquery用来把一个查询转换成tsquery数据类型。to_tsquery提供了比plainto_tsquery和phraseto_tsquery更多的特性，但是它对其输入要求更加严格。

```sql
to_tsquery([ config regconfig, ] querytext text) returns tsqueryz
```

*可以被附加到一个词位来指定前缀匹配：

```sql
SELECT to_tsquery('supern:*A & star:A*B');
to_tsquery
--------------------------
'supern':*A & 'star':*AB
```

### 排名搜索结果

​	排名处理尝试度量文档和一个特定查询的接近程度，这样当有很多匹配时最相关的那些可以被先显示。PostgreSQL提供了两种预定义的排名函数，它们考虑词法、临近性和结构信息；即，它们考虑查询词在文档中出现得有多频繁，文档中的词有多接近，以及词出现的文档部分有多重要。不过，相关性的概念是模糊的并且与应用非常相关。不同的应用可能要求额外的信息用于排名，例如，文档修改时间。内建的排名函数只是例子。你可以编写你自己的排名函数和/或把它们的结果与附加因素整合在一起来适应你的特定需求。

目前可用的两种排名函数是：

```sql
ts_rank([ weights float4[], ] vector tsvector, query tsquery [, normalization integer ]) returns float4
```

基于向量的匹配词位的频率来排名向量。

```sql
ts_rank_cd([ weights float4[], ] vector tsvector, query tsquery [, normalization integer ]) returns float4
```

对这两个函数，可选的权重参数提供了为词实例赋予更多或更少权重的能力，这种能力是依据它们被标注的情况的。权重数组指定每一类词应该得到多重的权重，按照如下的顺序：
{D-权重, C-权重, B-权重, A-权重}
如果没有提供权重，那么将使用这些默认值：
{0.1, 0.2, 0.4, 1.0}



由于一个较长的文档有更多的机会包含一个查询术语，因此考虑文档的尺寸是合理的，例如一个一百个词的文档中有一个搜索词的五个实例而零一个一千个词的文档中有该搜索词的五个实例，则前者比后者更相关。两种排名函数都采用一个整数正规化选项，它指定文档长度是否影响其排名以及如何影响。该整数选项控制多个行为，因此它是一个位掩码：你可以使用|指定一个或多个行为（例如，2|4）。

* 0（默认值）忽略文档长度
* 1 用 1 + 文档长度的对数除排名
* 2 用文档长度除排名
* 4 用长度之间的平均调和距离除排名（只被ts_rank_cd实现）
* 8 用文档中唯一词的数量除排名
* 16 用 1 + 文档中唯一词数量的对数除排名
* 32 用排名 + 1 除排名

```sql
SELECT title, ts_rank_cd(textsearch, query, 32 /* rank/(rank+1) */ ) AS rank
FROM apod, to_tsquery('neutrino|(dark & matter)') query
WHERE query @@ textsearch
ORDER BY rank DESC
LIMIT 10;
title | rank
-----------------------------------------------+-------------------
Neutrinos in the Sun | 0.756097569485493
The Sudbury Neutrino Detector | 0.705882361190954
A MACHO View of Galactic Dark Matter | 0.668123210574724
Hot Gas and Dark Matter | 0.65655958650282
The Virgo Cluster: Hot Plasma and Dark Matter | 0.656301290640973
Rafting for Solar Neutrinos | 0.655172410958162
NGC 4650A: Strange Galaxy and Dark Matter | 0.650072921219637
Hot Gas and Dark Matter | 0.617195790024749
Ice Fishing for Cosmic Neutrinos | 0.615384618911517
Weak Lensing Distorts the Universe | 0.450010798361481
```



### 加亮结果

​	要表示搜索结果，理想的方式是显示每一个文档的一个部分并且显示它是怎样与查询相关的。通常，搜索引擎显示文档片段时会对其中的搜索术语进行标记。PostgreSQL提供了一个函数ts_headline来实现这个功能。

```sql
ts_headline([ config regconfig, ] document text, query tsquery [, options text ]) returns text
```

## 额外特性

### 操纵文档

略

### 操纵查询

略

#### 查询重写

略

### 用于自动更新的触发器

​	当使用一个单独的列来存储你的文档的tsvector表示时，有必要创建一个触发器在文档内容列改变时更新tsvector列。两个内建触发器函数可以用于这个目的，或者你可以编写你自己的触发器函数。

```sql
tsvector_update_trigger(tsvector_column_name, config_name, text_column_name [, ... ])
tsvector_update_trigger_column(tsvector_column_name, config_column_name, text_column_name
[, ... ])
```

这些触发器函数在CREATE TRIGGER命令中指定的参数控制下，自动从一个或多个文本列计算一个tsvector列。它们使用的一个例子是：

```sql
CREATE TABLE messages (
title text,
body text,
tsv tsvector
);

CREATE TRIGGER tsvectorupdate BEFORE INSERT OR UPDATE
ON messages FOR EACH ROW EXECUTE PROCEDURE
tsvector_update_trigger(tsv, 'pg_catalog.english', title, body);
INSERT INTO messages VALUES('title here', 'the body text is here');
SELECT * FROM messages;
title | body | tsv
------------+-----------------------+----------------------------
title here | the body text is here | 'bodi':4 'text':5 'titl':1
SELECT title, body FROM messages WHERE tsv @@ to_tsquery('title & body');
title | body
------------+-----------------------
title here | the body text is here
```

​	在创建了这个触发器后，在title或body中的任何修改将自动地被反映到tsv中，不需要应用来操心同步的问题。

​	这些内建触发器的一个限制是它们将所有输入列同样对待。要对列进行不同的处理 — 例
如，使标题的权重和正文的不同 — 就需要编写一个自定义触发器。下面是用PL/pgSQL作为触发器语言的一个例子：

略

### 收集文档统计数据

ts_stat被用于检查你的配置以及寻找候选的停用词。

```sql
ts_stat(sqlquery text, [ weights text, ]
OUT word text, OUT ndoc integer,
OUT nentry integer) returns setof record
```

## 解析器

​	文本搜索解析器负责把未处理的文档文本划分成记号并且标识每一个记号的类型，而可能的类型集合由解析器本身定义。注意一个解析器完全不会修改文本 — 它简单地标识看似有理的词边界。因为这种有限的视野，对于应用相关的自定义解析器的需求就没有自定义字典那么强烈。目前PostgreSQL只提供了一种内建解析器，它已经被证实对很多种应用都适用。

## 词典

![](https://pic.imgdb.cn/item/6269358e239250f7c522bff3.jpg)

### 停用词

略

### 简单词典

​	simple词典模板的操作是将输入记号转换为小写形式并且根据一个停用词文件检查它。如果该记号在该文件中被找到，则返回一个空数组，导致该记号被丢弃。否则，该词的小写形式被返回作为正规化的词位。作为一种选择，该词典可以被配置为将非停用词报告为未识别，允许它们被传递给列表中的下一个词典。

### 同义词词典

​	这个词典模板被用来创建用于同义词替换的词典。不支持短语（使用分类词典模板
（第 12.6.4 节）可以支持）。一个同义词词典可以被用来解决语言学问题，例如，阻止一
个英语词干分析器词典把词“Paris”缩减成“pari”。在同义词词典中有一行Paris paris并把它放在english_stem词典之前就足够了。例如：

### 分类词典

​	一个分类词典（有时被简写成TZ）是一个词的集合，其中包括了词与短语之间的联系，即广义词（BT）、狭义词（NT）、首选词、非首选词、相关词等。

### 分类词典配置

要定义一个新的分类词典，可使用thesaurus模板。例如：
```sql
CREATE TEXT SEARCH DICTIONARY thesaurus_simple (
TEMPLATE = thesaurus,
DictFile = mythesaurus,
Dictionary = pg_catalog.english_stem
);
```

### Ispell 词典

​	Ispell词典模板支持词法词典，它可以把一个词的很多不同语言学的形式正规化成相同的词位。例如，一个英语Ispell词典可以匹配搜索词bank的词尾变化和词形变化，例如banking、banked、banks、banks'和bank's。

# 并发控制

## 事务隔离

**脏读**
一个事务读取了另一个并行未提交事务写入的数据。
**不可重复读**
一个事务重新读取之前读取过的数据，发现该数据已经被另一个事务（在初始读之后提
交）修改。

**幻读**
一个事务重新执行一个返回符合一个搜索条件的行集合的查询， 发现满足条件的行集合
因为另一个最近提交的事务而发生了改变。

**序列化异常**
成功提交一组事务的结果与这些事务所有可能的串行执行结果都不一致。

![](https://pic.imgdb.cn/item/626950ab239250f7c5639d55.jpg)

### 读已提交隔离级别

​	读已提交是PostgreSQL中的默认隔离级别。 当一个事务运行使用这个隔离级别时， 一个查询（没有FOR UPDATE/SHARE子句）只能看到查询开始之前已经被提交的数据， 而无法看到未提交的数据或在查询执行期间其它事务提交的数据。实际上，SELECT查询看到的是一个在查询开始运行的瞬间该数据库的一个快照。不过SELECT可以看见在它自身事务中之前执行的更新的效果，即使它们还没有被提交。还要注意的是，即使在同一个事务里两个相邻的SELECT命令可能看到不同的数据， 因为其它事务可能会在第一个SELECT开始和第二个SELECT开始之间提交。

​	UPDATE、DELETE、SELECT FOR UPDATE和SELECT FOR SHARE命令在搜索目标行时的
行为和SELECT一样： 它们将只找到在命令开始时已经被提交的行。 不过，在被找到时，这样的目标行可能已经被其它并发事务更新（或删除或锁住）。在这种情况下， 即将进行的更新将等待第一个更新事务提交或者回滚（如果它还在进行中）。 如果第一个更新事务回滚，那么它的作用将被忽略并且第二个事务可以继续更新最初发现的行。 如果第一个更新事务提交，若该行被第一个更新者删除，则第二个更新事务将忽略该行，否则第二个更新者将试图在该行的已被更新的版本上应用它的操作。该命令的搜索条件（WHERE子句）将被重新计算来看该行被更新的版本是否仍然符合搜索条件。如果符合，则第二个更新者使用该行已更新版本继续其操作。在SELECT FOR UPDATE和SELECT FOR SHARE的情况下，这意味着把该行的已更新版本锁住并返回给客户端。

### 可重复读隔离级别

​	可重复读隔离级别只看到在事务开始之前被提交的数据；它从来看不到未提交的数据或者并行事务在本事务执行期间提交的修改（不过，查询能够看见在它的事务中之前执行的更新，即使它们还没有被提交）。这是比SQL标准对此隔离级别所要求的更强的保证，并且阻止表 13.1中描述的除了序列化异常之外的所有现象。如上面所提到的，这是标准特别允许的，标准只描述了每种隔离级别必须提供的最小保护。

### 可序列化隔离级别

​	可序列化隔离级别提供了最严格的事务隔离。这个级别为所有已提交事务模拟序列事务执行；就好像事务被按照序列一个接着另一个被执行，而不是并行地被执行。但是，和可重复读级别相似，使用这个级别的应用必须准备好因为序列化失败而重试事务。事实上，这个给力级别完全像可重复读一样地工作，除了它会监视一些条件，这些条件可能导致一个可序列化事务的并发集合的执行产生的行为与这些事务所有可能的序列化（一次一个）执行不一致。这种监控不会引入超出可重复读之外的阻塞，但是监控会产生一些负荷，并且对那些可能导致序列化异常的条件的检测将触发一次序列化失败。

​	要保证真正的可序列化，PostgreSQL使用了谓词锁，这意味着它会保持锁，这些锁让它能够判断在它先运行的情况下，什么时候一个写操作会对一个并发事务中之前读取的结果产生影响。在PostgreSQL中，这些锁并不导致任何阻塞，并且因此不会导致一个死锁。它们被用来标识和标志并发可序列化事务之间的依赖性，这些事务的组合可能导致序列化异常。相反，一个想要保证数据一致性的读已提交或可重复读事务可能需要拿走一个在整个表上的锁，这可能阻塞其他尝试使用该表的用户，或者它可能会使用不仅会阻塞其他事务还会导致磁盘访问的SELECT FOR UPDATE或SELECT FOR SHARE。

当依赖可序列化事务进行并发控制时，为了最佳性能应该考虑一下问题：

* 在可能时声明事务为READ ONLY。
* 控制活动连接的数量，如果需要使用一个连接池。这总是一个重要的性能考虑，但是在一个使用可序列化事务的繁忙系统中这尤为重要。
* 只在一个单一事务中放完整性目的所需要的东西。
* 不要让连接不必要地“闲置在事务中”。配置参数idle_in_transaction_session_timeout可以被用来自动断开拖延会话的连接。
* 在那些由于使用可序列化事务自动提供的保护的地方消除不再需要的显式锁、SELECT
  FOR UPDATE和SELECT FOR SHARE。
* 当系统因为谓词锁表内存短缺而被强制结合多个页面级谓词锁为一个单一的关系级谓词锁时，序列化失败的比例可能会上升。你可以通过增加max_pred_locks_per_transaction、max_pred_locks_per_relation和max_pred_locks_per_page来避免这种情况。
* 一次顺序扫描将总是需要一个关系级谓词锁。这可能导致序列化失败的比例上升。通过缩减random_page_cost和/或增加cpu_tuple_cost来鼓励使用索引扫描将有助于此。一定要在事务回滚和重启数目的任何减少与查询执行时间的任何全面改变之间进行权衡。

## 显式锁定

### 表级锁

下面的列表显示了可用的锁模式和PostgreSQL自动使用它们的场合。 你也可以用LOCK命令显式获得这些锁。请记住所有这些锁模式都是表级锁，即使它们的名字包含“row”单词（这些名称是历史遗产）。 在一定程度上，这些名字反应了每种锁模式的典型用法 — 但
是语意却都是一样的。 两种锁模式之间真正的区别是它们有着不同的冲突锁模式集合（参
考表 13.2）。 两个事务在同一时刻不能在同一个表上持有属于相互冲突模式的锁（但是，一个事务决不会和自身冲突。例如，它可以在同一个表上获得ACCESS EXCLUSIVE锁然后接
着获取ACCESS SHARE锁）。非冲突锁模式可以由许多事务同时持有。 请特别注意有些锁
模式是自冲突的（例如，在一个时刻ACCESS EXCLUSIVE锁不能被多于一个事务持有)而其
他锁模式不是自冲突的（例如，ACCESS SHARE锁可以被多个事务持有)。

**ACCESS SHARE**
只与ACCESS EXCLUSIVE锁模式冲突。
SELECT命令在被引用的表上获得一个这种模式的锁。通常，任何只读取表而不修改它的查询都将获得这种锁模式。

**ROW SHARE**

与EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。
SELECT FOR UPDATE和SELECT FOR SHARE命令在目标表上取得一个这种模式的锁 （加上在被引用但没有选择FOR UPDATE/FOR SHARE的任何其他表上的ACCESS SHARE锁）。

**ROW EXCLUSIVE**

​	与SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。

​	命令UPDATE、DELETE和INSERT在目标表上取得这种锁模式（加上在任何其他被引用表上的ACCESS SHARE锁）。通常，这种锁模式将被任何修改表中数据的命令取得。

**SHARE UPDATE EXCLUSIVE**

​	与SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW、EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。这种模式保护一个表不受并发模式改变和VACUUM运行的影响。

​	由VACUUM（不带FULL）、ANALYZE、CREATE INDEX CONCURRENTLY、CREATE STATISTICS和ALTER TABLE VALIDATE以及其他ALTER TABLE的变体获得。

**SHARE**
	与ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE ROW EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。这种模式保护一个表不受并发数据改变的影响。
	由CREATE INDEX（不带CONCURRENTLY）取得。

**SHARE ROW EXCLUSIVE**

​	与ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。这种模式保护一个表不受并发数据修改所影响，并且是自排他的，这样在一个时刻只能有一个会话持有它。
​	由CREATE COLLATION、CREATE TRIGGER和很多 ALTER TABLE的很多形式所获得（见 ALTER TABLE）。

**EXCLUSIVE**
	与ROW SHARE、ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE锁模式冲突。这种模式只允许并发的ACCESS SHARE锁，即只有来自于表的读操作可以与一个持有该锁模式的事务并行处理。
	由REFRESH MATERIALIZED VIEW CONCURRENTLY获得。

**ACCESS EXCLUSIVE**

​	与所有模式的锁冲突（ACCESS SHARE、ROW SHARE、ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE和ACCESS EXCLUSIVE）。这种模式保证持有者是访问该表的唯一事务。

​	由ALTER TABLE、DROP TABLE、TRUNCATE、REINDEX、CLUSTER、VACUUM FULL和REFRESH MATERIALIZED VIEW（不带CONCURRENTLY）命令获 取。ALTER TABLE的很多形式也在这个层面上获得锁（见ALTER TABLE）。这也是未显式指定模式的LOCK TABLE命令的默认锁模式。

![](https://pic.imgdb.cn/item/626960e2239250f7c58f228a.jpg)

### 行级锁

**FOR UPDATE**

​	FOR UPDATE会导致由SELECT语句检索到的行被锁定，就好像它们要被更新。这可以
阻止它们被其他事务锁定、修改或者删除，一直到当前事务结束。也就是说其他尝试UPDATE、DELETE、SELECT FOR UPDATE、SELECT FOR NO KEY UPDATE、SELECT FOR SHARE或者SELECT FOR KEY SHARE这些行的事务将被阻塞，直到当前事务结束。反过来，SELECT FOR UPDATE将等待已经在相同行上运行以上这些命令的并发事务，并且接着锁定并且返回被更新的行（或者没有行，因为行可能已被删除）。不过，在一个REPEATABLE READ或SERIALIZABLE事务中，如果一个要被锁定的行在事务开始后被更改，将会抛出一个错误。进一步的讨论请见第 13.4 节。

​	任何在一行上的DELETE命令也会获得FOR UPDATE锁模式，在某些列上修改值的UPDATE也会获得该锁模式。当前UPDATE情况中被考虑的列集合是那些具有能用于外键的唯一索引的列（所以部分索引和表达式索引不被考虑），但是这种要求未来有可能会改变。

**FOR NO KEY UPDATE**

​	行为与FOR UPDATE类似，不过获得的锁较弱：这种锁将不会阻塞尝试在相同行上获得锁的SELECT FOR KEY SHARE命令。任何不获取FOR UPDATE锁的UPDATE也会获得这种锁模式。

**FOR SHARE**

​	行为与FOR NO KEY UPDATE类似，不过它在每个检索到的行上获得一个共享锁而不是排他锁。一个共享锁会阻塞其他事务在这些行上执行UPDATE、DELETE、SELECT FORUPDATE或者SELECT FOR NO KEY UPDATE，但是它不会阻止它们执行SELECT FOR SHARE或者SELECT FOR KEY SHARE。

**FOR KEY SHARE**

​	行为与FOR SHARE类似，不过锁较弱：SELECT FOR UPDATE会被阻塞，但是SELECT
FOR NO KEY UPDATE不会被阻塞。一个键共享锁会阻塞其他事务执行修改键值的DELETE或者UPDATE，但不会阻塞其他UPDATE，也不会阻止SELECT FOR NO KEY UPDATE、SELECT FOR SHARE或者SELECT FOR KEY SHARE。

![](https://pic.imgdb.cn/item/62696165239250f7c5915fb2.jpg)

### 页级锁

​	除了表级别和行级别的锁以外，页面级别的共享/排他锁被用来控制对共享缓冲池中表页面的读/写。 这些锁在行被抓取或者更新后马上被释放。应用开发者通常不需要关心页级锁，我们在这里提到它们只是为了完整。

### 死锁

​	显式锁定的使用可能会增加死锁的可能性，死锁是指两个（或多个）事务相互持有对方想要的锁。例如，如果事务 1 在表 A 上获得一个排他锁，同时试图获取一个在表
B 上的排他锁， 而事务 2 已经持有表 B 的排他锁，同时却正在请求表A 上的一个排他锁，那么两个事务就都不能进行下去。PostgreSQL能够自动检测到死锁情况并且会通过中断其中一个事务从而允许其它事务完成来解决这个问题（具体哪个事务会被中断是很难预测的，而且也不应该依靠这样的预测）。

### 咨询锁

​	PostgreSQL提供了一种方法创建由应用定义其含义的锁。这种锁被称为咨询锁，因为系统并不强迫其使用 — 而是由应用来保证其正确的使用。咨询锁可用于 MVCC 模型不适用的锁定策略。例如，咨询锁的一种常用用法是模拟所谓“平面文件”数据管理系统典型的悲观锁策略。虽然一个存储在表中的标志可以被用于相同目的，但咨询锁更快、可以避免表膨胀并且会由服务器在会话结束时自动清理。

## 应用级别的数据完整性检查

​	对于使用读已提交事务的数据完整性强制业务规则非常困难，因为对每一个语句数据视图都在变化，并且如果一个写冲突发生即使一个单一语句也不能把它自己限制到该语句的快照。

### 用可序列化事务来强制一致性

略

### 使用显式锁定强制一致性

​	当可以使用非可序列化写时，要保证一行的当前有效性并保护它不受并发更新的影响，我们必须使用SELECT FOR UPDATE、SELECT FOR SHARE或一个合适的LOCK TABLE 语句（SELECT FOR UPDATE和SELECT FOR SHARE锁只针对并发更新返回行，而LOCK TABLE会锁住整个表）。当从其他环境移植应用到PostgreSQL时需要考虑这些。

## 提醒

​	一些 DDL 命令（当前只有TRUNCATE和表重写形式的ALTER TABLE）对于MVCC 不是安全的。这意味着在截断或者重写提交之后，该表将对并发事务（如果它们使用的快照是在 DDL 命令提交前取得的）呈现出空表的形态

## 锁定和索引

​	尽管PostgreSQL提供对表数据访问的非阻塞读/写， 但并非PostgreSQL中实现的每一个索引访问方法当前都能够提供非阻塞读/写访问。 不同的索引类型按照下面方法操作：

**B-tree、GiST和SP-GiST索引**

​	短期的页面级共享/排他锁被用于读/写访问。每个锁银行被取得或被插入后立即释放锁。 这些索引类型提供了无死锁情况的最高并发性。

**Hash索引**

​	Hash 桶级别的共享/排他锁被用于读/写访问。锁在整个 Hash 桶处理完成后释放。Hash 桶级锁比索引级的锁提供了更好的并发性但是可能产生死锁，因为锁持有的时间比一次索引操作时间长。

**GIN索引**

​	短期的页面级共享/排他锁被用于读/写访问。 锁在索引行被插入/抓取后立即释放。但要注意的是一个 GIN 索引值的插入通常导致对每行产生几个索引键的插入，因此 GIN 可能为了插入一个单一值而做大量的工作。

​	目前，B-tree 索引为并发应用提供了最好的性能。因为它还有比 Hash 索引更多的特性，在那些需要对标量数据进行索引的并发应用中，我们建议使用 B-tree 索引类型。在处理非标量类型数据的时候，B-tree 就没什么用了，应该使用 GiST、SP-GiST 或 GIN 索引替代。

# 性能提示

## 使用EXPLAIN

### EXPLAIN基础

​	查询计划的结构是一个计划结点的树。最底层的结点是扫描结点：它们从表中返回未经处理的行。 不同的表访问模式有不同的扫描结点类型：顺序扫描、索引扫描、位图索引扫描。也还有不是表的行来源，例如VALUES子句和FROM中返回集合的函数，它们有自己的结点类型。如果查询需要连接、聚集、排序、或者在未经处理的行上的其它操作，那么就会在扫描结点之上有其它额外的结点来执行这些操作。 并且，做这些操作通常都有多种方法，因此在这些位置也有可能出现不同的结点类型。 EXPLAIN给计划树中每个结点都输出一行，显示基本的结点类型和计划器为该计划结点的执行所做的开销估计。 第一行（最上层的结点）是对该计划的总执行开销的估计；计划器试图最小化的就是这个数字

```sqk
EXPLAIN SELECT * FROM tenk1;
QUERY PLAN
-------------------------------------------------------------
Seq Scan on tenk1 (cost=0.00..458.00 rows=10000 width=244)
```

由于这个查询没有WHERE子句，它必须扫描表中的所有行，因此计划器只能选择使用一个简单的顺序扫描计划。被包含在圆括号中的数字是（从左至右）：

* 估计的启动开销。在输出阶段可以开始之前消耗的时间，例如在一个排序结点里执行排序的时间。
* 估计的总开销。这个估计值基于的假设是计划结点会被运行到完成，即所有可用的行都被检索。不过实际上一个结点的父结点可能很快停止读所有可用的行（见下面的LIMIT例子）。
* 这个计划结点输出行数的估计值。同样，也假定该结点能运行到完成。
* 预计这个计划结点输出的行平均宽度（以字节计算）。

开销是用规划器的开销参数（参见第 19.7.2 节）所决定的捏造单位来衡量的。传统上以取磁盘页面为单位来度量开销； 也就是seq_page_cost将被按照习惯设为1.0，其它开销参数将相对于它来设置。 本节的例子都假定这些参数使用默认值。

这些数字的产生非常直接。如果你执行：

```sql
SELECT relpages, reltuples FROM pg_class WHERE relname = 'tenk1';
```

你会发现tenk1有358个磁盘页面和10000行。 开销被计算为 

`（页面读取数 * seq_page_cost）+（扫描的行数*cpu_tuple_cost）`。默认情况下，seq_page_cost是 1.0，cpu_tuple_cost是0.01， 因此估计的开销是 (358 * 1.0) + (10000 * 0.01) = 458。

现在让我们修改查询并增加一个WHERE条件：
```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 7000;

QUERY PLAN
------------------------------------------------------------

Seq Scan on tenk1 (cost=0.00..483.00 rows=7001 width=244)
Filter: (unique1 < 7000)
```

请注意EXPLAIN输出显示WHERE子句被当做一个“过滤器”条件附加到顺序扫描计划结
点。 这意味着该计划结点为它扫描的每一行检查该条件，并且只输出通过该条件的行。因
为WHERE子句的存在，估计的输出行数降低了。不过，扫描仍将必须访问所有 10000 行，因此开销没有被降低；实际上开销还有所上升（准确来说，上升了 10000 * cpu_operator_cost）以反映检查WHERE条件所花费的额外 CPU 时间。

​	这条查询实际选择的行数是 7000，但是估计的行数只是个近似值。如果你尝试重复这个试验，那么你很可能得到略有不同的估计。 此外，这个估计会在每次ANALYZE命令之后改变， 因为ANALYZE生成的统计数据是从该表中随机采样计算的。

```sql
# 现在，让我们把条件变得更严格：
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 100;
QUERY PLAN
------------------------------------------------------------------------------
Bitmap Heap Scan on tenk1 (cost=5.07..229.20 rows=101 width=244)
Recheck Cond: (unique1 < 100)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..5.04 rows=101 width=0)
Index Cond: (unique1 < 100)
```

### EXPLAIN ANALYZE

​	可以通过使用EXPLAIN的ANALYZE选项来检查规划器估计值的准确性。通过使用这个选
项，EXPLAIN会实际执行该查询，然后显示真实的行计数和在每个计划结点中累计的真实运行时间，还会有一个普通EXPLAIN显示的估计值。例如，我们可能得到这样一个结果：

```sql
EXPLAIN ANALYZE SELECT *
FROM tenk1 t1, tenk2 t2
WHERE t1.unique1 < 10 AND t1.unique2 = t2.unique2;
QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------
Nested Loop (cost=4.65..118.62 rows=10 width=488) (actual time=0.128..0.377 rows=10 loops=1)
-> Bitmap Heap Scan on tenk1 t1 (cost=4.36..39.47 rows=10 width=244) (actual
time=0.057..0.121 rows=10 loops=1)
Recheck Cond: (unique1 < 10)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..4.36 rows=10 width=0) (actual
time=0.024..0.024 rows=10 loops=1)
Index Cond: (unique1 < 10)
-> Index Scan using tenk2_unique2 on tenk2 t2 (cost=0.29..7.91 rows=1 width=244) (actual
time=0.021..0.022 rows=1 loops=10)
Index Cond: (unique2 = t1.unique2)
Planning time: 0.181 ms
Execution time: 0.501 ms
```

​	注意“actual time”值是以毫秒计的真实时间，而cost估计值被以捏造的单位表示，因此它们不大可能匹配上。在这里面要查看的最重要的一点是估计的行计数是否合理地接近实际值。在这个例子中，估计值都是完全正确的，但是在实际中非常少见。



```sql
EXPLAIN ANALYZE SELECT * FROM polygon_tbl WHERE f1 @> polygon '(0.5,2.0)';
QUERY PLAN
------------------------------------------------------------------------------------------------------
Seq Scan on polygon_tbl (cost=0.00..1.05 rows=1 width=32) (actual time=0.044..0.044 rows=0
loops=1)
Filter: (f1 @> '((0.5,2))'::polygon)
Rows Removed by Filter: 4
Planning time: 0.040 ms
Execution time: 0.083 ms
```



​	规划器认为（非常正确）这个采样表太小不值得劳烦一次索引扫描，因此我们得到了一个普通的顺序扫描，其中的所有行都被过滤器条件拒绝。但是如果我们强制使得一次索引扫描可以被使用，我们看到：

```sql
SET enable_seqscan TO off;
EXPLAIN ANALYZE SELECT * FROM polygon_tbl WHERE f1 @> polygon '(0.5,2.0)';
QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------
Index Scan using gpolygonind on polygon_tbl (cost=0.13..8.15 rows=1 width=32) (actual
time=0.062..0.062 rows=0 loops=1)
Index Cond: (f1 @> '((0.5,2))'::polygon)
Rows Removed by Index Recheck: 1
Planning time: 0.034 ms
Execution time: 0.144 ms
```

这里我们可以看到索引返回一个候选行，然后它会被索引条件的重新检查拒绝。这是因为一个 GiST 索引对于多边形包含测试是 “有损的”：它确实返回覆盖目标的多边形的行，然后我们必须在那些行上做精确的包含性测试。



EXPLAIN有一个BUFFERS选项可以和ANALYZE一起使用来得到更多运行时统计信息：

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM tenk1 WHERE unique1 < 100 AND
unique2 > 9000;
QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------
Bitmap Heap Scan on tenk1 (cost=25.08..60.21 rows=10 width=244) (actual time=0.323..0.342
rows=10 loops=1)
Recheck Cond: ((unique1 < 100) AND (unique2 > 9000))
Buffers: shared hit=15
-> BitmapAnd (cost=25.08..25.08 rows=10 width=0) (actual time=0.309..0.309 rows=0 loops=1)
Buffers: shared hit=7
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..5.04 rows=101 width=0) (actual
time=0.043..0.043 rows=100 loops=1)
Index Cond: (unique1 < 100)
Buffers: shared hit=2
-> Bitmap Index Scan on tenk1_unique2 (cost=0.00..19.78 rows=999 width=0) (actual
time=0.227..0.227 rows=999 loops=1)
Index Cond: (unique2 > 9000)
Buffers: shared hit=5
Planning time: 0.088 ms
Execution time: 0.423 ms
```

​	BUFFERS提供的数字帮助我们标识查询的哪些部分是对 I/O 最敏感的。

​	记住因为EXPLAIN ANALYZE实际运行查询，任何副作用都将照常发生，即使查询可能输出的任何结果被丢弃来支持打印EXPLAIN数据。如果你想要分析一个数据修改查询而不想改变你的表，你可以在分析完后回滚命令，例如：

```sql
BEGIN;
EXPLAIN ANALYZE UPDATE tenk1 SET hundred = hundred + 1 WHERE unique1 < 100;
QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------
Update on tenk1 (cost=5.07..229.46 rows=101 width=250) (actual time=14.628..14.628 rows=0
loops=1)
-> Bitmap Heap Scan on tenk1 (cost=5.07..229.46 rows=101 width=250) (actual
time=0.101..0.439 rows=100 loops=1)
Recheck Cond: (unique1 < 100)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..5.04 rows=101 width=0) (actual
time=0.043..0.043 rows=100 loops=1)
Index Cond: (unique1 < 100)
Planning time: 0.079 ms
Execution time: 14.727 ms
ROLLBACK;
```

### 警告

​	在两种有效的方法中EXPLAIN ANALYZE所度量的运行时间可能偏离同一个查询的正常执行。首先，由于不会有输出行被递交给客户端，网络传输开销和 I/O 转换开销没有被包括在内。其次，由EXPLAIN ANALYZE所增加的度量符合可能会很可观，特别是在那些gettimeofday()操作系统调用很慢的机器上。你可以使用pg_test_timing工具来度量在你的系统上的计时开销。

​	EXPLAIN结果不应该被外推到与你实际测试的非常不同的情况。例如，一个很小的表上的结果不能被假定成适合大型表。规划器的开销估计不是线性的，并且因此它可能为一个更大或更小的表选择一个不同的计划。一个极端例子是，在一个只占据一个磁盘页面的表上，你将几乎总是得到一个顺序扫描计划，而不管索引是否可用。规划器认识到它在任何情况下都将采用一次磁盘页面读取来处理该表，因此用额外的页面读取去查看一个索引是没有价值的（我们已经在前面的polygon_tbl例子中见过）。

​	在一些情况中，实际的值和估计的值不会匹配得很好，但是这并非错误。一种这样的情况发生在计划结点的执行被LIMIT或类似的效果很快停止。例如，在我们之前用过的LIMIT查询中：

```sql
EXPLAIN ANALYZE SELECT * FROM tenk1 WHERE unique1 < 100 AND unique2 > 9000
LIMIT 2;
QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------
Limit (cost=0.29..14.71 rows=2 width=244) (actual time=0.177..0.249 rows=2 loops=1)
-> Index Scan using tenk1_unique2 on tenk1 (cost=0.29..72.42 rows=10 width=244) (actual
time=0.174..0.244 rows=2 loops=1)
Index Cond: (unique2 > 9000)
Filter: (unique1 < 100)
Rows Removed by Filter: 287
Planning time: 0.096 ms
Execution time: 0.336 ms
```

## 规划器使用的统计信息

### 单列统计

​	如我们在上一节所见，查询规划器需要估计一个查询要检索的行数，这样才能对查询计划做出好的选择。 本节对系统用于这些估计的统计信息进行一个快速的介绍。

​	统计信息的一个部分就是每个表和索引中的项的总数，以及每个表和索引占用的磁盘块数。这些信息保存在pg_class表的reltuples和relpages列中。 我们可以用类似下面的查询查看这些信息：

```sql
SELECT relname, relkind, reltuples, relpages
FROM pg_class
WHERE relname LIKE 'tenk1%';
relname | relkind | reltuples | relpages
----------------------+---------+-----------+----------
tenk1 | r | 10000 | 358
tenk1_hundred | i | 10000 | 30
tenk1_thous_tenthous | i | 10000 | 30
tenk1_unique1 | i | 10000 | 30
tenk1_unique2 | i | 10000 | 30
(5 rows)
```

​	这里我们可以看到tenk1包含 10000 行， 它的索引也有这么多行，但是索引远比表小得多（不奇怪）。

​	出于效率考虑，reltuples和relpages不是实时更新的 ，因此它们通常包含有些过时的值。它们被VACUUM、ANALYZE和几个 DDL 命令（例如CREATE INDEX）更新。一个不扫描全表的VACUUM或ANALYZE操作（常见情况）将以它扫描的部分为基础增量更新reltuples计数，这就导致了一个近似值。在任何情况中，规划器将缩放它在pg_class中找到的值来匹配当前的物理表尺寸，这样得到一个较紧的近似。

​	大多数查询只是检索表中行的一部分，因为它们有限制要被检查的行的WHERE子句。 因此规划器需要估算WHERE子句的选择度，即符合WHERE子句中每个条件的行的比例。 用于这个任务的信息存储在pg_statistic系统目录中。 在pg_statistic中的项由ANALYZE和VACUUM ANALYZE命令更新， 并且总是近似值（即使刚刚更新完）。

​	除了直接查看pg_statistic之外， 手工检查统计信息的时候最好查看它的视图pg_stats。pg_stats被设计为更容易阅读。 而且，pg_stats是所有人都可以读取的，而pg_statistic只能由超级用户读取（这样可以避免非授权用户从统计信息中获取一些其他人的表的内容的信息。pg_stats视图被限制为只显示当前用户可读的表）。例如，我们可以：

```sql
SELECT attname, inherited, n_distinct,
array_to_string(most_common_vals, E'\n') as most_common_vals
FROM pg_stats
WHERE tablename = 'road';
attname | inherited | n_distinct | most_common_vals
---------+-----------+------------+------------------------------------
name | f | -0.363388 | I- 580 Ramp+
| | | I- 880 Ramp+
| | | Sp Railroad +
| | | I- 580 +
| | | I- 680 Ramp
name | t | -0.284859 | I- 880 Ramp+
| | | I- 580 Ramp+
| | | I- 680 Ramp+
| | | I- 580 +
| | | State Hwy 13 Ramp
(2 rows)
```

​	注意，这两行显示的是相同的列，一个对应开始于road表（inherited=t）的完全继承层次，另一个只包括road表本身（inherited=f）。

​	ANALYZE在pg_statistic中存储的信息量（特别是每个列的most_common_vals中的最大项数和histogram_bounds数组）可以用ALTER TABLE SET STATISTICS命令为每一列设置， 或者通过设置配置变量default_statistics_target进行全局设置。 目前的默认限制是 100 个项。提升该限制可能会让规划器做出更准确的估计（特别是对那些有不规则数据分布的列）， 其代价是在pg_statistic中消耗了更多空间，并且需要略微多一些的时间来计算估计数值。 相比之下，比较低的限制可能更适合那些数据分布比较简单的列。

### 扩展统计

​	通常会看到缓慢的查询运行错误的执行计划，因为查询子句中使用的多列是相关的。 规划器通常假定多个条件彼此独立，当列值相关时，这种假设不成立。 由于每个单独列的性质，定期统计无法捕捉有关跨列关联的任何知识。 但是，PostgreSQL能够计算多元统计信息， 它可以捕获这些信息。

​	由于可能的列组合数量非常大，因此自动计算多元统计信息是不切实际的。 相反，可以创建扩展统计信息对象， 通常称为统计信息对象，以指示服务器通过有趣的列集获得统计信息。

​	统计信息对象是使用CREATE STATISTICS创建的， 它可以查看更多详细信息。创建这样一个对象只是创建一个表示对统计信息感兴趣的目录条目。 实际的数据收集由ANALYZE执行（手动命令或后端自动分析）。 可以在pg_statistic_ext 目录中检查收集的值。

#### 函数依赖

最简单的扩展统计信息跟踪函数依赖，一个用于数据库标准表单的定义中的概念。 如果知
道a的值足够确定b的值， 我们说列b函数依赖于列a， 即没有两行会有相同的a值但b值不同。在完全规范化的数据库中，函数依赖关系只应存在于主键和超级键上。 但是，实际上很多数据集由于各种原因未完全标准化； 出于性能原因的故意的非规范化是一个常见的例子。即使在完全标准化的数据库中， 某些列之间也可能存在部分相关性，可以将其表示为部分函数依赖性。

**函数依赖的局限性**

​	函数依赖性当前仅在考虑将列与常量值进行比较的简单相等条件时才适用。 它们不用于改进对比较两列或将列与表达式进行比较的相等条件的估计值， 也不用于范围子句、LIKE或任何其他类型的条件。

#### N 个不同多变量的计数

单列统计信息存储每列中不同值的数量。当规划器只有单列统计数据时， 如果组合多个列
（例如，对于GROUP BY a, b）， 对不同值个数的估计常常是错误的，导致它选择不好的计划。

## 用显式JOIN子句控制规划器

​	我们可以在一定程度上用显式JOIN语法控制查询规划器。要明白为什么需要它，我们首先需要一些背景知识。

```sql
SELECT * FROM a, b, c WHERE a.id = b.id AND b.ref = c.id;
```

​	规划器可以自由地按照任何顺序连接给定的表。例如，它可以生成一个使用WHERE条件a.id= b.id连接 A 到 B 的查询计划，然后用另外一个WHERE条件把 C 连接到这个连接表。或者它可以先连接 B 和 C 然后再连接 A 得到同样的结果。 或者也可以连接 A 到 C 然后把结果与 B连接 — 不过这么做效率不好，因为必须生成完整的 A 和 C 的迪卡尔积，而在WHERE子句中没有可用条件来优化该连接（PostgreSQL执行器中的所有连接都发生在两个输入表之间， 所以它必须以这些形式之一建立结果）。 重要的一点是这些不同的连接可能性给出在语义等效的结果，但在执行开销上却可能有巨大的差别。 因此，规划器会对它们进行探索并尝试找出最高效的查询计划。

​	当一个查询只涉及两个或三个表时，那么不需要考虑很多连接顺序。但是可能的连接顺序数随着表数目的增加成指数增长。 当超过十个左右的表以后，实际上根本不可能对所有可能性做一次穷举搜索，甚至对六七个表都需要相当长的时间进行规划。 当有太多的输入表时，PostgreSQL规划器将从穷举搜索切换为一种遗传概率搜索，它只需要考虑有限数量的可能性（切换的阈值用geqo_threshold运行时参数设置）。遗传搜索用时更少，但是并不一定会找到最好的计划。

​	当查询涉及外连接时，规划器比处理普通（内）连接时拥有更小的自由度。例如，考虑：

```sql
SELECT * FROM a LEFT JOIN (b JOIN c ON (b.ref = c.id)) ON (a.id = b.id);
```

尽管这个查询的约束表面上和前一个非常相似，但它们的语义却不同， 因为如果 A 里
有任何一行不能匹配 B 和 C的连接表中的行，它也必须被输出。因此这里规划器对连接顺序没有什么选择：它必须先连接 B 到 C，然后把 A 连接到该结果上。 相应地，这个查询比前面一个花在规划上的时间更少。在其它情况下，规划器就有可能确定多种连接顺序都是安全的。例如，给定：

```sql
SELECT * FROM a LEFT JOIN b ON (a.bid = b.id) LEFT JOIN c ON (a.cid = c.id);
```

​	将 A 首先连接到 B 或 C 都是有效的。当前，只有FULL JOIN完全约束连接顺序。大多数涉及LEFT JOIN或RIGHT JOIN的实际情况都在某种程度上可以被重新排列。

​	显式连接语法（INNER JOIN、CROSS JOIN或无修饰的JOIN）在语义上和FROM中列出输入关系是一样的， 因此它不约束连接顺序。

​	要强制规划器遵循显式JOIN的连接顺序， 我们可以把运行时参数join_collapse_limit设置为1（其它可能值在下文讨论)。

​	按照这种方法约束规划器的搜索是一个有用的技巧，不管是对减少规划时间还是对引导规划器生成好的查询计划。 如果规划器按照默认选择了一个糟糕的连接顺序，你可以通过JOIN语法强迫它选择一个更好的顺序 — 假设你知道一个更好的顺序。我们推荐进行实验。

## 填充一个数据库

​	第一次填充数据库时可能需要插入大量的数据。本节包含一些如何让这个处理尽可能高效的建议。

### 禁用自动提交

略

### 使用COPY

​	使用COPY在一条命令中装载所有记录，而不是一系列INSERT命令。 COPY命令是为装载大量行而优化过的； 它没INSERT那么灵活，但是在大量数据装载时导致的负荷也更少。 因为COPY是单条命令，因此使用这种方法填充表时无须关闭自动提交。

### 移除索引

​	如果你正在载入一个新创建的表，最快的方法是创建该表，用COPY批量载入该表的数据，然后创建表需要的任何索引。在已存在数据的表上创建索引要比在每一行被载入时增量地更新它更快。

### 移除外键约束

​	和索引一样，“成批地”检查外键约束比一行行检查效率更高。 因此，先删除外键约束、载入数据然后重建约束会很有用。 同样，载入数据和约束缺失期间错误检查的丢失之间也存在平衡。

### 增加maintenance_work_mem

​	在载入大量数据时，临时增大maintenance_work_mem配置变量可以改进性能。这个参数也可以帮助加速CREATE INDEX命令和ALTER TABLE ADD FOREIGN KEY命令。 它不会对COPY本身起很大作用，所以这个建议只有在你使用上面的一个或两个技巧时才有用。

### 增加max_wal_size

临时增大max_wal_size配置变量也可以让大量数据载入更快。 这是因为向PostgreSQL中载入大量的数据将导致检查点的发生比平常（由checkpoint_timeout配置变量指定）更频繁。无论何时发生一个检查点时，所有脏页都必须被刷写到磁盘上。 通过在批量数据载入时临时增加max_wal_size，所需的检查点数目可以被缩减。

### 禁用 WAL 归档和流复制

​	当使用 WAL 归档或流复制向一个安装中载入大量数据时，在录入结束后执行一次新的基础备份比处理大量的增量 WAL 数据更快。为了防止载入时记录增量 WAL，通过将wal_level设置为minimal、将archive_mode设置为off以及将max_wal_senders设置为零来禁用归档和流复制。 但需要注意的是，修改这些设置需要重启服务。

### 事后运行ANALYZE

​	不管什么时候你显著地改变了表中的数据分布后，我们都强烈推荐运行ANALYZE。着包括向表中批量载入大量数据。运行ANALYZE（或者VACUUM ANALYZE）保证规划器有表的最新统计信息。 如果没有统计数据或者统计数据过时，那么规划器在查询规划时可能做出很差劲决定，导致在任意表上的性能低下。需要注意的是，如果启用了 autovacuum 守护进程，它可能会自动运行ANALYZE；参阅第 24.1.3 节和第 24.1.6 节。

### 关于pg_dump的一些注记

​	pg_dump生成的转储脚本自动应用上面的若干个（但不是全部）技巧。 要尽可能快地载入pg_dump转储，你需要手工做一些额外的事情（请注意，这些要点适用于恢复一个转储，而不是创建它的时候。同样的要点也适用于使用psql载入一个文本转储或用pg_restore从一个pg_dump归档文件载入）。

​	默认情况下，pg_dump使用COPY，并且当它在生成一个完整的模式和数据转储时， 它会很小心地先装载数据，然后创建索引和外键。因此在这种情况下，一些指导方针是被自动处理的。你需要做的是：

* 为maintenance_work_mem和max_wal_size设置适当的（即比正常值大的）值。
* 如果使用 WAL 归档或流复制，在转储时考虑禁用它们。在载入转储之前，可通过
  将archive_mode设置为off、将wal_level设置为minimal以及将max_wal_senders设置为零（在录入dump前）来实现禁用。 之后，将它们设回正确的值并执行一次新的基础备份。
* 采用pg_dump和pg_restore的并行转储和恢复模式进行实验并且找出要使用的最佳并发任务数量。通过使用-j选项的并行转储和恢复应该能为你带来比串行模式高得多的性能。
* 考虑是否应该在一个单一事务中恢复整个转储。要这样做，将-1或--single-transaction命令行选项传递给psql或pg_restore。 当使用这种模式时，即使是一个很小的错误也会回滚整个恢复，可能会丢弃已经处理了很多个小时的工作。根据数据间的相关性， 可能手动清理更好。如果你使用一个单一事务并且关闭了 WAL 归档，COPY命令将运行得最快。
* 如果在数据库服务器上有多个 CPU 可用，可以考虑使用pg_restore的--jobs选项。这允许并行数据载入和索引创建。
* 之后运行ANALYZE。



​	一个只涉及数据的转储仍将使用COPY，但是它不会删除或重建索引，并且它通常不会触碰外键。 1 因此当载入一个只有数据的转储时，如果你希望使用那些技术，你需要负责删除并重建索引和外键。在载入数据时增加max_wal_size仍然有用，但是不要去增加maintenance_work_mem；不如说在以后手工重建索引和外键时你已经做了这些。并且不要忘记在完成后执行ANALYZE，详见第 24.1.3 节和第 24.1.6 节。

## 非持久设置

​	持久性是数据库的一个保证已提交事务的记录的特性（即使是发生服务器崩溃或断电）。 然而，持久性会明显增加数据库的负荷，因此如果你的站点不需要这个保证，PostgreSQL可以被配置成运行更快。在这种情况下，你可以调整下列配置来提高性能。除了下面列出的，在数据库软件崩溃的情况下也能保证持久性。当这些设置被使用时，只有突然的操作系统停止会产生数据丢失或损坏的风险。

* 将数据库集簇的数据目录放在一个内存支持的文件系统上（即RAM磁盘）。这消除了所有的数据库磁盘 I/O，但将数据存储限制到可用的内存量（可能有交换区）。
* 关闭fsync；不需要将数据刷入磁盘。
* 关闭synchronous_commit；可能不需要在每次提交时 强制把WAL写入磁盘。这种设置可能会在 数据库崩溃时带来事务丢失的风险（但是没有数据破坏）。
* 关闭full_page_writes；不许要警惕部分页面写入。
* 增加max_wal_size和checkpoint_timeout； 这会降低检查点的频率，但会 增加/pg_wal的存储要求。
* 创建不做日志的表 来避免WAL写入，不过这会让表在崩溃时不安全。

# 并行查询

​	PostgreSQL能设计出利用多 CPU 让查询更快的查询计划。这种特性被称为并行查询。由于现有实现的限制或者因为没有比连续查询计划更快的查询计划存在，很多查询并不能从并行查询获益。不过，对于那些可以从并行查询获益的查询来说，并行查询带来的速度提升是显著的。很多查询在使用并行查询时比之前快了超过两倍，有些查询是以前的四倍甚至更多的倍数。那些访问大量数据但只返回其中少数行给用户的查询最能从并行查询中获益。这一章介绍一些并行查询如何工作的细节以及哪些情况下可以使用并行查询，这样希望充分利用并行查询的用户可以理解他们能从并行查询得到什么。

## 并行查询如何工作

​	当优化器判断对于某一个特定的查询，并行查询是最快的执行策略时，优化器将创建一个查询计划。该计划包括一个Gather或Gather Merge节点。下面是一个简单的例子：

```sql
EXPLAIN SELECT * FROM pgbench_accounts WHERE filler LIKE '%x%';
QUERY PLAN
-------------------------------------------------------------------------------------
Gather (cost=1000.00..217018.43 rows=1 width=97)
Workers Planned: 2
-> Parallel Seq Scan on pgbench_accounts (cost=0.00..216018.33 rows=1 width=97)
Filter: (filler ~~ '%x%'::text)
(4 rows)
```

​	在所有的情形下，Gather或Gather Merge 节点都只有一个子计划，它是将被并行执行的计划的一部分。如果 Gather 或Gather Merge节点位于计划树的最顶层，那么整个查询将并行执行。 如果它位于计划树的其他位置，那么只有在它下面的计划部分会并行执行。 在上面的例子中，查询只访问了一个表，因此除Gather 节点本身之外只有一个计划节点。因为该计划节点是 Gather 节点的孩子节点，所以它会并行执行。

使用 EXPLAIN命令, 你能看到规划器选择的工作者数量。 当查询执行期间到达Gather节点
时， 实现用户会话的进程将会请求和规划器选中的工作者数量一样多的 后台工作者进程 。

规划器考虑使用的后台工作者的数量限制为最多 max_parallel_workers_per_gather。 任何时候能够存在的后台工作者进程的总数由max_worker_processes 和max_parallel_workers限制， 因此一个并行查询可能会使用比规划中少的工作者来运行， 甚至有可能根本不使用工作者。最优的计划可能取决于可用的工作者的数量， 因此这可能会导致不好的查询性能。如果这种情况经常发生， 那么就应当考虑一下提高max_worker_processes和max_parallel_workers 的值，这样更多的工作者可以同时运行；或者降低max_parallel_workers_per_gather， 这样规划器会要求少一些的工作者。

## 何时会用到并行查询？

有几种设置会导致查询规划器在任何情况下都不生成并行查询计划。为了让并行查询计划能够被生成，必须配置好下列设置。

* max_parallel_workers_per_gather必须被设置为大于零的值。这是一种特殊情况，更加普遍的原则是所用的工作者数量不能超过max_parallel_workers_per_gather所配置的数量。
* dynamic_shared_memory_type必须被设置为除none之外的值。并行查询要求动态共享内存以便在合作的进程之间传递数据。



​	此外，系统一定不能运行在单用户模式下。因为在单用户模式下，整个数据库系统运行在单个进程中，没有后台工作者进程可用。

​	如果下面的任一条件为真，即便对一个给定查询通常可以产生并行查询计划，规划器都不会为它产生并行查询计划：

* 查询要写任何数据或者锁定任何数据库行。如果一个查询在顶层或者 CTE 中包含了数据修改操作，那么不会为该查询产生并行计划。这是当前实现的一个限制，未来的版本中可能会有所改进。
* 查询可能在执行过程中被暂停。只要在系统认为可能发生部分或者增量式执行，就不会产生并行计划。例如：用DECLARE CURSOR创建的游标将永远不会使用并行计划。类似地，一个FOR x IN query LOOP .. END LOOP形式的 PL/pgSQL 循环也永远 不会使用并行计划，因为当并行查询进行时，并行查询系统无法验证循环中的代码执行起来是安全的。
* 使用了任何被标记为PARALLEL UNSAFE的函数的查询。大多数系统定义的函数都被标记为PARALLEL SAFE，但是用户定义的函数默认被标记为PARALLEL UNSAFE。参
  见第 15.4 节中的讨论。
* 该查询运行在另一个已经存在的并行查询内部。例如，如果一个被并行查询调用的函数自己发出一个 SQL 查询，那么该查询将不会使用并行计划。这是当前实现的一个限制，但是或许不值得移除这个限制，因为它会导致单个查询使用大量的进程。
* 事务隔离级别是可串行化。这是当前实现的一个限制。



即使对于一个特定的查询已经产生了并行查询计划，在一些情况下执行时也不会并行执行该计划。如果发生这种情况，那么领导者将会自己执行该计划在Gather节点之下的部分，就好像Gather节点不存在一样。上述情况将在满足下面的任一条件时发生：

* 因为后台工作者进程的总数不能超过max_worker_processes，导致不能得到后台工作者进程。
* 由于为并行查询而启动的后台工作者总数不能超过 max_parallel_workers，因此无法获得后台工作者。
* 客户端发送了一个执行消息，并且消息中要求取元组的数量不为零。执行消息可见扩展查询协议中的讨论。因为libpq当前没有提供方法来发送这种消息，所以这种情况只可能发生在不依赖 libpq 的客户端中。如果这种情况经常发生，那在它可能发生的会话中将max_parallel_workers_per_gather设置为0是一个很好的主意，这样可以避免产生连续运行时次优的查询计划。
* 准备好的语句是使用CREATE TABLE .. AS EXECUTE ..语句执行的。 此构造将本来只是只读操作的内容转换为读写操作，使其不适合并行查询。
* 事务隔离级别是可串行化。这种情况通常不会出现，因为当事务隔离级别是可串行化时不会产生并行查询计划。不过，如果在产生计划之后并且在执行计划之前把事务隔离级别改成可串行化，这种情况就有可能发生。

## 并行计划

​	因为每个工作者只执行完成计划的并行部分，所以不可能简单地产生一个普通查询计划并使用多个工作者运行它。每个工作者都会产生输出结果集的一个完全拷贝，因而查询并不会比普通查询运行得更快甚至还会产生不正确的结果。相反，计划的并行部分一定被查询优化器在内部当作一个部分计划。也就是说，一定要这样来创建计划，使得每个将执行该计划的进程只产生输出行的一个子集，这样可以保证每个需要被输出的行刚好会被合作进程产生一次。 通常，这意味着查询驱动表上的扫描必须是并行感知扫描。

### 并行扫描

目前支持以下类型的并行感知表扫描。

* 在并行顺序扫描中，表格的块将在协作过程中分配。 块一次发放一个，所以访问表仍然是连续的。
* 在一个并行位图堆扫描中，选择一个进程作为领导者。 该进程执行一个或多个索引的扫描并构建指示需要访问哪些表块的位图。 然后这些块在并行顺序扫描中在协作进程中分配。
  换句话说， 堆扫描是并行执行的，但底层索引扫描不是。
* 在并行索引扫描或并行仅索引的扫描中， 协作进程轮流从索引读取数据。目前， 并行索引扫描仅支持btree索引。每个进程将声明一个索引块， 并将扫描并返回该块引用的所有元组；其他进程可以同时从不同的索引块返回元组。 并行btree扫描的结果以每个工作进程内的排序顺序返回。
  其他扫描类型（如非Btree索引的扫描）可能会在将来支持并行扫描。

### 并行连接

​	就像在非平行计划中一样，可以使用嵌套循环、 散列连接或合并连接将驱动表连接到一个或多个其他表。 连接的内侧可以是规划器支持的任何类型的非平行计划， 只要在并行工作者内运行是安全的。例如，如果选择了嵌套循环连接， 则内部计划可能是索引扫描，它会查找从连接外侧获取的值。

### 并行聚合

​	PostgreSQL通过两个阶段的聚合来支持并行聚合。第一次， 每个参与查询计划并行部分执行的进程执行一个聚合步骤， 为进程发现的每个分组产生一个部分结果。这在计划中反映为一个 PartialAggregate节点。第二次，部分结果被通过Gather 或Gather Merge节点传输给领导者。最后， 领导者对所有工作者的部分结果进行重聚合以得到最终的结果。 这在计划中反映为一个FinalizeAggregate节点。

### 并行计划小贴士

​	如果我们想要一个查询能产生并行计划但事实上又没有产生，可以尝试减小parallel_setup_cost或者parallel_tuple_cost。当然，这个计划可能比规划器优先产生的顺序计划还要慢，但也不总是如此。如果将这些设置为很小的值（例如把它们设置为零）也不能得到并行计划，那就可能是有某种原因导致查询规划器无法为你的查询产生并行计划。可能的原因可见第 15.2 节和第 15.4 节。

## 并行安全性

规划器把查询中涉及的操作分类成并行安全、并行受限 或者并行不安全。

下面的操作总是并行受限的。
• 公共表表达式（CTE）的扫描。
• 临时表的扫描。
• 外部表的扫描，除非外部数据包装器有一个IsForeignScanParallelSafe API。
• 对InitPlan或者相关的SubPlan的访问。

### 为函数和聚合加并行标签

规划器无法自动判定一个用户定义的函数或者聚合是并行安全、并行受限还是并行不安全，因为这需要预测函数可能执行的每一个操作。一般而言，这就相当于一个停机问题，因此是不可能的。甚至对于可以做到判定的简单函数我们也不会尝试，因为那会非常昂贵而且容易出错。相反，除非是被标记出来，所有用户定义的函数都被认为是并行不安全的。在使用CREATE FUNCTION或者ALTER FUNCTION时，可以通过指定PARALLEL
SAFE、PARALLEL RESTRICTED或者PARALLEL UNSAFE来设置标记 。在使用CREATE
AGGREGATE时，PARALLEL选项可以被指定为SAFE、RESTRICTED或者 UNSAFE。



​	如果函数和聚合会写数据库、访问序列、改变事务状态（即便是临时改变，例如建立一
个EXCEPTION块来捕捉错误的 PL/pgSQL）或者对设置做持久化的更改，它们一定要被标记为PARALLEL UNSAFE。类似地，如果函数会访问临时表、客户端连接状态、游标、预备语句或者系统无法在工作者之间同步的后端本地状态，它们必须被标记为PARALLEL
RESTRICTED。例如，setseed和 random由于后一种原因而是并行受限的。
