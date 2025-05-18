# 1. 简单查询

## 1.1 查询一个字段

>查询一个字段就是是：一个表有多列，查询其中的一列，SQL 语句使用大小写都可以，但一条SQL语句必须以 `;` 结尾（在 MySQL 命令行客户端中）

```sql
语法格式:
select 字段名 from 表名;
```

例如：查询公司中所有员工编号

```sql
select ename from emp; 
```

![](images/DQL%20数据查询语言/file-20250518160229.png)

****
## 1.2 查询多个字段

>查询多个字段时，在字段名和字段名之间添加 `,` 即可

```sql
语法格式:
select 字段名1,字段名2,字段名3 from 表名;
```

例如：查询员工编号以及员工姓名

```sql
select empno, ename from emp;
```

![](images/DQL%20数据查询语言/file-20250518160431.png)

****
## 1.3 查询所有字段

>查询所有字段可以将每个字段都列出来查询，也可以采用 `*` 来代表所有字段

例如：查询员工的所有信息

```sql
select * from emp;
```

![](images/DQL%20数据查询语言/file-20250518160547.png)

### 实际开发中不推荐使用 `*` 

> `select * ` 在实际执行时会先将语句解析成对应的所有字段，然后再把数据返回，这一步骤可能会导致更多的时间和资源的消耗，并且可读性更差，不能直到到底查询了什么

****
## 1.4 查询的字段可参与运算

例如：查询每个员工的年薪（月薪 * 12）

```sql
select ename, sal * 12 from emp;
```

![](images/DQL%20数据查询语言/file-20250518161446.png)

****
## 1.5 起别名

>使用关键字 `as` 可以给某个字段进行重命名，在实际展示的时候会以新的名字作为列名

```sql
select ename, sal * 12 as yearsal from emp;
```

![](images/DQL%20数据查询语言/file-20250518161630.png)

>不过 `as` 关键字通常可以省略，中间使用空格代替，但别名中间不能有空格，这会让某些关键字查询不到导致编译错误

```sql
select ename, sal * 12 yearsal from emp;
```

****
# 2. 条件查询

| **条件**              | **说明**                              |
| ------------------- | ----------------------------------- |
| =                   | 等于                                  |
| <> 或 !=             | 不等于                                 |
| >=                  | 大于等于                                |
| <=                  | 小于等于                                |
| >                   | 大于                                  |
| <                   | 小于                                  |
| BETWEEN ... AND ... | 在某个范围内（包括边界），等同于 >= 和 <= 的组合        |
| IS NULL             | 值为空（NULL）                           |
| IS NOT NULL         | 值不为空                                |
| <=>                 | 安全等于（NULL-safe 等于），用于判断 NULL 相等，极少用 |
| AND 或 &&            | 逻辑“并且”                              |
| OR 或 \|             | 逻辑“或者”                              |
| IN                  | 值在指定集合中                             |
| NOT IN              | 值不在指定集合中                            |
| EXISTS              | 子查询结果集非空时为真（用于判断子查询是否有数据）           |
| NOT EXISTS          | 子查询结果集为空时为真                         |
| LIKE                | 模糊匹配，通常配合 `%`（任意字符）和 `_`（单字符）       |

## 2.1 where 关键字

>`WHERE` 是 SQL 语句中用来指定筛选条件的关键字，它用来从表中筛选出满足条件的行（记录），只有满足 `WHERE` 条件的记录才会被查询、更新或删除，一般后面跟的字段都要加上 `''`
 
****
## 2.2 and 和 or 的优先级

>在 SQL 中 and 的优先级高于 or，但使用时建议用 `()` 明确优先级

```sql
select
  ename,sal,deptno
from
  emp
where
  sal < 1500 and deptno = 20 or deptno = 30;

-- 系统解析为
(sal < 1500 AND deptno = 20) OR (deptno = 30)
```

![](images/DQL%20数据查询语言/file-20250518171721.png)

>薪资高于 1500 的也被查找出来了，所以应该把 sql 语句修改为：

```sql
SELECT
  ename, sal, deptno
FROM
  emp
WHERE
  sal < 1500 AND (deptno = 20 OR deptno = 30);
```

![](images/DQL%20数据查询语言/file-20250518171822.png)

****
## 2.3 between...and...

>用于判断值是否在某个区间范围内，包含边界，即等同于： `>= 最小值 AND <= 最大值` ，但必须是左小右大，与 `>= AND <=` 区别只是语法写法不同，底层原理与执行效率完全一致

```sql
表达式 BETWEEN 最小值 AND 最大值
```

```sql
SELECT 
  ename, sal
FROM
  emp
WHERE
  sal BETWEEN 3000 AND 1600;

-- 等于
sal >= 3000 AND sal <= 1600
```

查找不到数据

![](images/DQL%20数据查询语言/file-20250518172212.png)

****
## 2.4 in、not in

>`job in('MANAGER','SALESMAN','CLERK')` 等同于 `job = 'MANAGER' or job = 'SALESMAN' or job = 'CLERK'` ， `sal in(1600, 3000, 5000)` 等同于 `sal = 1600 or sal = 3000 or sal = 5000`

```sql
select
  ename,sal,job
from
  emp
where
  job in('MANAGER', 'SALESMAN');
```

![](images/DQL%20数据查询语言/file-20250518172811.png)

>`job not in('MANAGER','SALESMAN')` 等同于 `job <> 'MANAGER' and job <> 'SALESMAN'` ， `sal not in(1600, 5000)` 等同于 `sal <> 1600 and sal <> 5000`

```sql
select 
  ename,job
from
  emp
where
  job not in('MANAGER', 'SALESMAN');
```

![](images/DQL%20数据查询语言/file-20250518172928.png)
### in、not in 与 NULL

>在 SQL 中，`NULL` 表示“未知”或“没有值”，不是 0、不是空字符串，也不是布尔意义上的 false，所以`NULL = NULL` 是不成立的，任何与 `NULL` 进行比较的表达式结果都是 `UNKNOWN` ，只有 `IS NULL` 或 `IS NOT NULL` 可以判断 NULL 值

1、`IN` 忽略 `NULL`

```sql
SELECT * FROM emp WHERE comm IN (NULL, 300);
```

>这条语句的直观预期是：查询 `comm` 为 NULL 或 300 的数据，但实际执行后只会查询到 `comm = 300` 的记录，因为`IN (NULL, 300)` 实际等价于 `comm = NULL OR comm = 300`

2、`NOT IN` 与 `NULL`

```sql
SELECT * FROM emp WHERE comm NOT IN (NULL, 300);
```

>这条语句的直观预期是：查询 `comm` 不是 NULL，也不是 300 的记录，但实际执行后什么记录都没查询到，因为`NOT IN (NULL, 300)` 实际等价于 `comm <> NULL AND comm <> 300` ，由于 `comm <> NULL` 是 `UNKNOWN`，然后 `AND` 运算中只要有一个是 `UNKNOWN` 或 `FALSE`，整体就不是 `TRUE`，所以所有记录都被排除了

**正确处理 in、not in 和 null**

>如果要查找 `comm` 是 NULL 或 300 就使用 `SELECT * FROM emp WHERE comm IS NULL OR comm = 300;`，如果要查找 `comm` 不是 NULL 且不等于 300 就使用 `SELECT * FROM emp WHERE comm IS NOT NULL AND comm <> 300;`

****
## 2.5 





