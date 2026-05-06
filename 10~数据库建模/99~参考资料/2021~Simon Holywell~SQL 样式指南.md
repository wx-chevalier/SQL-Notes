```
原文地址：https://www.sqlstyle.guide/zh/
```

# SQL 样式指南

## 概述

你可以使用这套 SQL 样式指南来维护更加易读和维护的 SQL 代码。这套指南与其他指南的主要区别是为了保持与 PHP 类似的代码样式，并针对 MySQL 和 PostgreSQL 两种数据库进行了调整。

这套指南借鉴了 [http://www.sqlstyle.guide](http://www.sqlstyle.guide) 的内容。

## 一般原则

### 做什么

- 使用一致的、描述性的命名。
- 合理使用空格和缩进来增强可读性。
- 存储日期和时间应使用 `DATETIME` 类型（或 `TIMESTAMP`）。
- 所有数据库设计都应该符合第三范式，除非有特殊需求。
- 除非有性能考虑，否则所有字段应该声明为 `NOT NULL`。
- 除非有性能考虑，否则应该使用 `INNER JOIN` 而不是 `WHERE` 子句。
- 需要为所有存储过程参数指定 `IN/OUT`。

### 避免什么

- 驼峰命名法 - 这与大多数语言的命名习惯不同。
- 描述性前缀或匈牙利命名法。
- 复数形式 - 尽可能使用更通用的术语。
- 需要引号的标识符 - 如果你必须使用这样的标识符，请坚持使用双引号。
- 面向对象的设计原则不应该被盲目地应用到数据库设计中。

## 命名规范

### 通用

- 确保名字独一无二且不是保留字。
- 长度应控制在 30 个字符以内。
- 名字必须以字母开头，不能以下划线结尾。
- 名字只能使用字母、数字和下划线。
- 避免使用多个连续下划线。
- 使用下划线分隔名字中的单词。
- 避免使用缩写词，除非是广为人知的缩写。

```sql
SELECT first_name
  FROM staff_members;
```

### 表

- 使用集合名词或者更通用的术语。
- 不要使用 `tbl` 或其他类似的前缀或匈牙利命名法。
- 表不应该同它们的列同名。
- 避免把表的名字和其中的列名连接起来作为关系列的名字。

```sql
-- 好的例子
staff_members
staff_member_details

-- 不好的例子
staff
staff_member_staff_member_details
```

### 列

- 总是使用单数形式。
- 避免使用表名作为列的前缀。
- 主键列名应该简单使用 `id`。
- 外键列应该使用单数形式加上 `_id` 后缀。

```sql
-- 好的例子
first_name
staff_member_id
id

-- 不好的例子
staff_member_first_name
staff_memberid
staffmemberid
```

### 别名/相关名

- 应该与对象或者它们的子部分相关。
- 通常是对象名的第一个字母。
- 如果已经有相同的别名，那么在后面加上数字。

```sql
SELECT s.first_name
  FROM staff_members AS s;
```

### 存储过程

- 名字必须包含动词。
- 不要使用 `sp_` 或其他类似的前缀。

```sql
DELIMITER //

CREATE PROCEDURE process_staff_member()
BEGIN
    -- 处理逻辑
END //

DELIMITER ;
```

### 统一后缀

下列后缀有统一的意义，使用时应遵守这些约定：

- `_id` - 唯一标识符
- `_status` - 标识或状态
- `_total` - 总和或总数
- `_num` - 字段包含数值
- `_name` - 名字
- `_seq` - 包含序列
- `_date` - 包含日期
- `_tally` - 计数
- `_size` - 大小
- `_addr` - 地址（物理或虚拟）

## 查询语句

### 保留字

- 保留字总是大写，如 `SELECT`、`WHERE`。
- 最好避免使用保留字作为名字。
- 单词应该分开，不要缩写。

```sql
SELECT first_name
  FROM staff_members
 WHERE id = 2;
```

### 空白

#### 空格

应该使用空格：

- 在等号（=）前后
- 在逗号（,）后
- 在单引号（'）外围

```sql
SELECT first_name,
       last_name,
       age
  FROM staff_members
 WHERE first_name = 'John';
```

#### 换行和缩进

- 每个主要的 SQL 关键字应该在新的一行。
- 缩进应该保持一致（建议使用两个或四个空格）。
- 括号应该和前面的命令在同一行。

```sql
SELECT a.id,
       a.name,
       b.total
  FROM table_a AS a
  JOIN table_b AS b
    ON a.id = b.a_id
 WHERE a.status = 'active'
   AND b.total > 100
 GROUP BY a.id,
          a.name;
```

### 查询语句中 WHERE 子句的顺序

WHERE 子句应该遵循以下顺序：

1. 最重要的条件
2. 对性能影响最大的条件
3. 特定的条件

```sql
SELECT *
  FROM staff_members
 WHERE active = 1
   AND department_id = 12
   AND created_date >= '2020-01-01';
```

## 创建表

### 数据类型

- 尽可能使用最精确和最小的类型。
- 只在真正需要浮点数运算时才使用 `REAL` 或 `FLOAT` 类型。
- 使用 `DECIMAL` 存储货币数值。
- 使用 `DATETIME` 存储时间和日期。

### 默认值

- 默认值必须和列的类型匹配。
- 默认值必须跟在类型声明后面。

```sql
CREATE TABLE staff_members (
    id INT NOT NULL AUTO_INCREMENT,
    first_name VARCHAR(30) NOT NULL,
    birth_date DATE NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

### 约束

- 每个表都应该有一个主键。
- 选择有意义的列组合作为外键。
- 更新和删除规则应该在外键上明确定义。

```sql
CREATE TABLE staff_members (
    id INT NOT NULL AUTO_INCREMENT,
    department_id INT NOT NULL,
    PRIMARY KEY (id),
    CONSTRAINT fk_department
        FOREIGN KEY (department_id)
        REFERENCES departments(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

## 附录

### 保留字参考

一个 SQL 保留字的完整列表可以在这里找到：[MySQL 保留字](https://dev.mysql.com/doc/refman/8.0/en/keywords.html)
