
>MySQL行转列又叫做数据透视，将原本横向排列的数据透视成纵向排列的数据，进而进行计算、分析、展示等操作

| student | subject | score |
| ------- | ------- | ----- |
| 张三      | 数学      | 90    |
| 张三      | 语文      | 80    |
| 张三      | 英语      | 70    |
| 李四      | 数学      | 85    |
| 李四      | 语文      | 78    |
| 李四      | 英语      | 88    |

# 1. 使用 `CASE WHEN` + `GROUP BY`

```sql
SELECT 
  student,
  MAX(CASE WHEN subject = '数学' THEN score END) AS 数学,
  -- 如果当前行的 subject 是“数学”，就返回对应的 score
  -- 否则返回 NULL
  MAX(CASE WHEN subject = '语文' THEN score END) AS 语文,
  MAX(CASE WHEN subject = '英语' THEN score END) AS 英语
FROM score
GROUP BY student;
```

需要注意的是：因为每个学生对于每一门课最多只会有一条成绩记录，因此用 `MAX()` 或 `SUM()`、`MIN()` 其实效果都一样；`MAX()` 是为了聚合，不写聚合函数 MySQL 会报错，可以取出非 NULL 值中的最大值，即“该学生该门课的成绩”

| student | 数学  | 语文  | 英语  |
| ------- | --- | --- | --- |
| 张三      | 90  | 80  | 70  |
| 李四      | 85  | 78  | 88  |

****
# 2. 配合 `IF()` 函数（是 `CASE` 的简化形式）

```sql
SELECT 
  student,
  MAX(IF(subject = '数学', score, NULL)) AS 数学,
  -- 如果当前行的 subject 是数学，就返回对应的 score
  -- 否则返回 NULL
  MAX(IF(subject = '语文', score, NULL)) AS 语文,
  MAX(IF(subject = '英语', score, NULL)) AS 英语
FROM score
GROUP BY student;
```

>作用和 `CASE WHEN` 一样，只是写法更简洁

****
