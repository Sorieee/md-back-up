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