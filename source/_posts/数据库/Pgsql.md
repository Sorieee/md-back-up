```sql
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

