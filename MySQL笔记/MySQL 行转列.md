
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
  MAX(CASE WHEN subject = '语文' THEN score END) AS 语文,
  MAX(CASE WHEN subject = '英语' THEN score END) AS 英语
FROM score
GROUP BY student;
```



