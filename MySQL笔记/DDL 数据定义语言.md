
|操作|SQL 语句|作用说明|
|---|---|---|
|创建数据库|`CREATE DATABASE`|创建一个新数据库|
|删除数据库|`DROP DATABASE`|删除整个数据库及其内容|
|创建表|`CREATE TABLE`|创建一张新表|
|修改表结构|`ALTER TABLE`|添加、删除、修改字段等|
|删除表|`DROP TABLE`|删除表及其结构|
|重命名表|`RENAME TABLE`|修改表名|
|清空表数据|`TRUNCATE TABLE`|清空表数据但保留结构|

# 1. CREATE DATABASE

```sql
CREATE DATABASE mydb;
-- 创建数据库 mydb，使用默认字符集和排序规则

CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
-- 指定字符集和校对规则
```

# 2. DROP DATABASE

```sql
DROP DATABASE mydb;
-- 删除数据库及其中所有表和数据
```

# 3. CREATE TABLE

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    age INT,
    hire_date DATE,
    salary DECIMAL(10,2)
);
```

创建表时，可以定义：

- 字段名、数据类型
- 主键（`PRIMARY KEY`）
- 是否非空（`NOT NULL`）
- 默认值（`DEFAULT`）
- 自动递增（`AUTO_INCREMENT`）

# 4. DROP TABLE

```sql
DROP TABLE employee;
-- 永久删除表和数据，不可恢复
```

# 5. ALTER TABLE 修改表结构

1、添加列：

```sql
ALTER TABLE employee ADD COLUMN email VARCHAR(255);
```

2、修改列类型或名称：

```sql
ALTER TABLE employee MODIFY age SMALLINT;
ALTER TABLE employee CHANGE name full_name VARCHAR(150);
```

3、删除列：

```sql
ALTER TABLE employee DROP COLUMN email;
```

4、添加或删除主键：

```sql
ALTER TABLE employee ADD PRIMARY KEY(id);
ALTER TABLE employee DROP PRIMARY KEY;
```

# 6. RENAME TABLE 重命名表

```sql
RENAME TABLE employee TO staff;
```

# 7. TRUNCATE TABLE 清空表

```sql
TRUNCATE TABLE employee;
```

>删除所有数据但不影响表结构，比 `DELETE FROM employee;` 更高效（不会逐条删除）

