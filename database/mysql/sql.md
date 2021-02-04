> mysql8.0.18

## 一、数据库

### 创建

```mysql
CREATE DATABASE test_db;
```

### 删除

```mysql
DROP DATABASE test_db;
```

### 使用(切换)

```mysql
USE test_db;
```

### 列出所有数据库

```mysql
SHOW DATABASES;
```

## 二、表

### 创建

```mysql
CREATE TABLE `author` (
	author_id INT,
	PRIMARY KEY(author_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `article`(
	article_id INT AUTO_INCREMENT,
	title VARCHAR(100) NOT NULL,
	fk_author_id int,
	submit_date time,
	PRIMARY KEY (article_id),
	FOREIGN key (fk_author_id) REFERENCES author(author_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 删除

```mysql
DROP TABLE mytable;
```

### 修改

添加列
```mysql
ALTER TABLE article
ADD COLUMN article_type VARCHAR(10) NOT NULL;
```

删除列
```mysql
ALTER TABLE article
DROP COLUMN article_type;
```

修改列名称和数据类型
```mysql
ALTER TABLE article 
CHANGE article_type type VARCHAR(12);
```

修改列数据类型
```mysql
ALTER TABLE article 
MODIFY title VARCHAR(50);
```

修改表名
```mysql
ALTER TABLE article 
RENAME TO passage;
```

### 查看表结构

```mysql
DESC article;
```

## 三、INSERT

普通插入
```mysql
INSERT INTO mytable(col1, col2)
VALUES(val1, val2);
```

插入检索出来的数据
```mysql
INSERT INTO mytable1(col1, col2)
SELECT col1, col2
FROM mytable2;
```

将一个表的内容插入到一个新表
```mysql
CREATE TABLE newtable AS
SELECT * FROM oldtable;
```

## 四 、UPDATE

```mysql
UPDATE mytable
SET col = val
WHERE id = 1;
```

## 五、DELETE

```mysql
DELETE FROM mytable
WHERE id = 1;
```

TRUNCATE TABLE 可以清空表
```mysql
TRUNCATE TABLE mytable;
```

## 六 、   SELECT

DISTINCT
相同值只会出现一次。它作用于所有列，也就是说所有列的值都相同才算相同。
```mysql
SELECT DISTINCT col1, col2
FROM mytable;
```

LIMIT
限制返回的行数。可以有两个参数，第一个参数为起始行，从 0 开始；第二个参数为返回的总行数。

返回前 5 行：
```mysql
SELECT *
FROM mytable
LIMIT 5;
```

```mysql
SELECT *
FROM mytable
LIMIT 0, 5;
```

返回第 3 ~ 5 行：
```mysql
SELECT *
FROM mytable
LIMIT 2, 3;
```

## 七、排序

ASC ：升序（默认）
DESC ：降序

可以按多个列进行排序，并且为每个列指定不同的排序方式：

```mysql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

## 八、通配符

只能用于文本字段。

% 匹配 >=0 个任意字符；

_ 匹配 ==1 个任意字符；

[ ] 可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

使用 Like 来进行通配符匹配。

```mysql
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%'; -- 不以 A 和 B 开头的任意文本
```

不要滥用通配符，通配符位于开头处匹配会非常慢。

## 九、 分组

把具有相同的数据值的行放在同一组中。

可以对同一分组数据使用汇总函数进行处理，例如求分组数据的平均值等。

指定的分组字段除了能按该字段进行分组，也会自动按该字段进行排序。

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col;
```

GROUP BY 自动按分组字段进行排序，ORDER BY 也可以按汇总字段来进行排序。

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col
ORDER BY num;
```

WHERE 过滤行，HAVING 过滤分组，行过滤应当先于分组过滤。

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING num >= 2;
```

分组规定：

- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
- NULL 的行会单独分为一组；
- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

##  十、子查询

子查询中只能返回一个字段的数据。

可以将子查询的结果作为 WHRER 语句的过滤条件：

```mysql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);
```

## 十一、连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 WHERE。

连接可以替换子查询，并且比子查询的效率一般会更快。

可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表。

### 内连接

在MySQL中`JOIN`, `CROSS JOIN`, and `INNER JOIN`是一样的, 不加条件就是笛卡尔积

> [MySQL](https://dev.mysql.com/doc/refman/8.0/en/join.html)
>
> In MySQL, `JOIN`, `CROSS JOIN`, and `INNER JOIN` are syntactic equivalents (they can replace each other). In standard SQL, they are not equivalent. `INNER JOIN` is used with an `ON` clause, `CROSS JOIN` is used otherwise.

内连接又称等值连接，使用 `INNER JOIN` 关键字。

```mysql
SELECT A.value, B.value
FROM tablea AS A INNER JOIN tableb AS B
ON A.key = B.key;
```

可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。

```mysql
SELECT A.value, B.value
FROM tablea AS A, tableb AS B
WHERE A.key = B.key;
```

### 自然连接

自然连接是把同名列通过等值测试连接起来的，同名列可以有多个, 查询后同名列合并。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。

```mysql
SELECT A.value, B.value
FROM tablea AS A NATURAL JOIN tableb AS B;
```

### 外连接

外连接保留了没有关联的那些行。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行, 对应的右表数据显示null。

`LEFT/RIGHT OUTER JOIN` 可缩写为 `LEFT/RIGHT JOIN`

检索所有顾客的订单信息，包括还没有订单信息的顾客。

```sql
SELECT Customers.cust_id, Customer.cust_name, Orders.order_id
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```

在MySQL中, 右外连接会被转化为左外连接

> [MySQL](https://dev.mysql.com/doc/refman/5.6/en/outer-join-simplification.html)
>
> At the parser stage, queries with right outer join operations are converted to equivalent queries containing only left join operations. 

## 十二、组合查询

组合查询相当于集合的并操作, 对数据进行合并, `UNION`会去除重复的行, `UNION ALL` 则不会

每个查询必须包含相同的列、表达式和聚集函数。

只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```mysql
(SELECT id, name FROM A)
UNION ALL
(SELECT id, name FROM B) 
ORDER BY id;
```

## 十三、事务管理

基本术语：

- 事务（transaction）指一组 SQL 语句；
- 回退（rollback）指撤销指定 SQL 语句的过程；
- 提交（commit）指将未存储的 SQL 语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

不能回退 SELECT 语句，回退 SELECT 语句也没意义；也不能回退 CREATE 和 DROP 语句。

MySQL 的事务提交默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭，重新恢复隐式提交。

设置 autocommit 为 0 可以取消自动提交；autocommit 标记是针对每个连接而不是针对服务器的。

如果没有设置保留点，ROLLBACK 会回退到 START TRANSACTION 语句处；如果设置了保留点，并且在 ROLLBACK 中指定该保留点，则会回退到该保留点。

```mysql
SET AUTOCOMMIT = 0 | 1;

START TRANSACTION;
// ...
SAVEPOINT tag;
// ...
ROLLBACK TO tag;
// ...
COMMIT;
```

##  十四、权限管理

MySQL 的账户信息保存在 mysql 这个数据库中。

```mysql
USE mysql;
SELECT user FROM user;
```

### 创建账户

新创建的账户没有任何权限。

```mysql
CREATE USER username IDENTIFIED BY 'password';
```

### 修改账户名

```mysql
RENAME USER olduname TO newuname;
```

### 删除账户

```mysql
DROP USER username;
```

### 查看权限

```mysql
SHOW GRANTS FOR username;
```

### 授予权限

账户用 username@host 的形式定义，username@% 使用的是默认主机名。

```mysql
GRANT SELECT, INSERT ON mydatabase.* TO myuser;
```

### 删除权限

GRANT 和 REVOKE 可在几个层次上控制访问权限：

- 整个服务器，使用 GRANT ALL 和 REVOKE ALL；
- 整个数据库，使用 ON database.*；
- 特定的表，使用 ON database.table；
- 特定的列；
- 特定的存储过程。

```sql
REVOKE SELECT, INSERT ON mydatabase.* FROM username;
```

### 更改密码

```sql
ALTER USER username IDENTIFIED WITH mysql_native_password BY 'password';
```

## 十五、函数

```mysql
# 字符函数
concat(..str) # 拼接字符串
length(str) # 返回字节长度, 中文占3或4个字节
char_length(str) # 返回字符个数, 中文占1个
substring(str, start, length) # 起始从1开始
trim(str)
trim(x from str) # 去除两端的x
upper()
lower()
strcmp() # x小


# 数学函数
# 日期函数
# 流程控制函数


ifnull(column, "asdf") # 如果前一个为null, 则使用后一个默认值
max()
min()
# 排名
row_number() over(order by column asc) # 连续排名
rank() over(order by column asc) 	   # 相同跳跃排名
dense_rank() over(order by column asc) # 相同连续排名

database() # 返回当前使用的数据库
version()  # 返回mysql版本
user()	   # 返回登录用户名和登录ip
```

