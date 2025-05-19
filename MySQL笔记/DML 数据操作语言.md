| 操作   | 关键词      | 作用说明                      |
| ---- | -------- | ------------------------- |
| 增加数据 | `INSERT` | 向表中插入一行或多行数据              |
| 查询数据 | `SELECT` | 查询表中的数据（DQL，常视为 DML 的一部分） |
| 修改数据 | `UPDATE` | 修改表中已有的记录                 |
| 删除数据 | `DELETE` | 删除表中的一行或多行记录              |

# 1. INSERT

1、 插入一行

```sql
-- 插入一行
INSERT INTO student (id, name, age) VALUES (1, 'Tom', 20);
```

2、插入多行

```sql
INSERT INTO student (id, name, age) 
VALUES 
  (2, 'Jerry', 21),
  (3, 'Alice', 22);
```

3、从其他表复制插入

```sql
-- 把 student 表中符合条件（年龄大于 22 岁）的学生的 id 和 name 复制到 graduate_students 表中
INSERT INTO graduate_students (id, name)
SELECT id, name FROM student WHERE age > 22;
```

# 2. UPDATE 

1、修改单条记录

```sql
-- 把 id = 1 的学生年龄修改为 21
UPDATE student SET age = 21 WHERE id = 1;
```

2、修改多条记录

```sql
-- 把所有年龄小于 25 岁的学生的年龄 +1 
UPDATE student SET age = age + 1 WHERE age < 25;
```

3、修改多个字段

```sql
UPDATE student SET name = 'John', age = 23 WHERE id = 2;
```

>`WHERE` 条件是用来筛选的，否则会更新整张表

# 3. DELETE

1、删除指定记录

```sql
DELETE FROM student WHERE id = 3;
```

2、删除多条记录

```sql
DELETE FROM student WHERE age < 20;
```

3、删除所有记录（不是删除表）

```sql
DELETE FROM student;
```

>和 `TRUNCATE TABLE student;` 效果类似，但 `DELETE` 可以回滚，`TRUNCATE` 通常不能





