## 一. 数据库

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

## 二. 表

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

## 三. INSERT

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

## 四. 更新

```mysql
UPDATE mytable
SET col = val
WHERE id = 1;
```

## 五. 删除

```mysql
DELETE FROM mytable
WHERE id = 1;
```

TRUNCATE TABLE 可以清空表
```mysql
TRUNCATE TABLE mytable;
```

## 六. 查询