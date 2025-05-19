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
## 2.5 模糊查询

>在 SQL 中，模糊查询是指使用通配符对字符串字段进行部分匹配查询，常用的是 `LIKE` 和 `NOT LIKE` 操作符，结合 `%` 和 `_` 这两个通配符来实现

```sql
SELECT 列名
FROM 表名
WHERE 列名 LIKE '匹配模式';
```

| 通配符   | 含义                                   | 示例                   |
| ----- | ------------------------------------ | -------------------- |
| `%`   | 匹配任意数量的任意字符（可为0个）                    | `'A%'`：以 A 开头        |
| `_`   | 匹配任意一个字符                             | `'A_'`：A 开头，后跟任意一个字符 |
| `[]`  | 匹配指定范围内的任意一个字符（部分数据库支持，如 SQL Server） | `'[ae]'`：匹配 a 或 e    |
| `[^]` | 排除指定范围的任意一个字符（部分数据库支持）               | `'[^ae]'`：排除 a 和 e   |


1、查找以 **S** 开头的员工姓名：

```sql
SELECT ename
FROM emp
WHERE ename LIKE 'S%';
```

![](images/DQL%20数据查询语言/file-20250518175903.png)

2、查找以 **N** 结尾的员工姓名

```sql
SELECT ename
FROM emp
WHERE ename LIKE '%N';
```

![](images/DQL%20数据查询语言/file-20250518175946.png)

3、查找姓名中包含 **A** 的员工

```sql
SELECT ename
FROM emp
WHERE ename LIKE '%A%';
```

>`%` 可匹配任意个字符，因此 `%A%` 表示 A 出现在任意位置

![](images/DQL%20数据查询语言/file-20250518180018.png)

4、查找第二个字母是 **L** 的员工姓名

```sql
SELECT ename
FROM emp
WHERE ename LIKE '_L%';
```

>`_` 匹配一个字符，`_L%` 表示第二个字符是 L

![](images/DQL%20数据查询语言/file-20250518180134.png)

5、查找不以 **S** 开头的员工姓名

```sql
SELECT ename
FROM emp
WHERE ename NOT LIKE 'S%';
```

![](images/DQL%20数据查询语言/file-20250518180232.png)

### 转义字符（ESCAPE）

如果你要查询内容中本身包含通配符字符，比如 `%` 或 `_`，需要使用 `ESCAPE` 来定义转义符：

```sql
SELECT *
FROM remarks
WHERE comment LIKE '%95!%%' ESCAPE '!';
```

>`!%` 表示字面上的 `%` 字符，`ESCAPE '!'` 定义 `!` 是转义字符，所以查找的记录是 `comment` 中带有 `95%` 的

****
# 3. 排序操作

```sql
SELECT 字段列表 FROM 表名
[WHERE 条件]
ORDER BY 排序字段 [ASC|DESC];
```

- `ORDER BY`：用于指定排序字段。

- `ASC`：升序（Ascending），从小到大。**默认方式**。

- `DESC`：降序（Descending），从大到小。

## 3.1 单字段升序

1、升序

>`ASC` 可省略，`ORDER BY sal` 效果一样

```sql
SELECT empno, ename, sal FROM emp ORDER BY sal ASC;
```

2、降序

```sql
SELECT empno, ename, sal FROM emp ORDER BY sal DESC;
```

****
## 3.2 多字段排序

>当排序字段有重复值时，可以进一步指定第二个字段排序规则：先按 `deptno` 升序排序，如果同部门编号员工有多名，则再按 `sal` 降序排序

```sql
SELECT empno, ename, sal, deptno
FROM emp
ORDER BY deptno ASC, sal DESC;
```

![](images/DQL%20数据查询语言/file-20250518183233.png)

****
## 3.3 null 的排序

>在 MySQL 中默认升序排序时 null 在最前面，降序排序时 null 在最后面

****
# 4. distinct去重

>对指定的所有列组合进行去重，保留唯一行，但 `DISTINCT` 必须放在 `SELECT` 和列名之间，也就是所有字段的最前面

```sql
SELECT DISTINCT 列名1, 列名2, ...
FROM 表名
[WHERE 条件];
```

![](images/DQL%20数据查询语言/file-20250518191308.png)

****
# 5. 数据处理函数

## 5.1 字符串相关

1、转大写 upper 和 ucase

```sql
-- 查询所有员工名字，以大写形式展现
select upper(ename) as ename from emp;

select ucase(ename) as ename from emp;
```

2、转小写 lower 和 lcase

```sql
-- 查询员工姓名，以小写形式展现
select lower(ename) as ename from emp;
select lcase(ename) as ename from emp;
```

3、截取字符串 substr

```sql
-- substr('被截取的字符串', 起始下标, 截取长度)，未指定截取长度时默认截取到字符串末尾
-- 找出员工名字中第二个字母是A的
select ename from emp where substr(ename, 2, 1) = 'A';
```

![](images/DQL%20数据查询语言/file-20250518193925.png)

4、获取字符串长度 length

```sql
select length('你好hello 123');
```

>可以看出一个汉字占三个长度

![](images/DQL%20数据查询语言/file-20250518194127.png)

5、获取字符的个数 char_length

```sql
select char_length('你好hello 123');
```

![](images/DQL%20数据查询语言/file-20250518194438.png)

6、字符串拼接

```sql
-- concat('字符串1', '字符串2', '字符串3'....)
select concat('zhangsan', 'lisi', 'wangwu');
```

![](images/DQL%20数据查询语言/file-20250518194721.png)

>需要注意的是 sql 语句中不能使用 `+` 拼接，它只会被识别为加法运算符，将加号两边的数据尽最大的努力转换成数字再求和，如果无法转换成数字，最终运算结果通通是 0

```sql
select 'zhangsan' + 'lisi';
```

![](images/DQL%20数据查询语言/file-20250518194919.png)

7、去除字符串前后空白 trim

```sql
select concat(trim('    abc    '), 'def');
```

![](images/DQL%20数据查询语言/file-20250518195337.png)

默认是去除前后空白，也可以去除指定的前缀后缀，例如：去除前置 0

```sql
select trim(leading '0' from '000111000');
```

![](images/DQL%20数据查询语言/file-20250518195439.png)

去除后置 0 

```sql
select trim(trailing '0' from '000111000');
```

![](images/DQL%20数据查询语言/file-20250518195508.png)

前置0和后置0全部去除

```sql
select trim(both '0' from '000111000');
```

![](images/DQL%20数据查询语言/file-20250518195540.png)

****
## 5.2 数字相关

1、rand() 和 rand(x)

rand() 生成 0 到 1 的随机浮点数

```sql
select rand();
```

![](images/DQL%20数据查询语言/file-20250518200336.png)

rand(x) 生成 0 到 1 的随机浮点数，通过指定整数 x 来确定每次获取到相同的浮点值

```sql
select rand(100);
```

2、round(x) 和 round(x,y) 四舍五入

round(x) 四舍五入，保留整数位，舍去所有小数

```sql
select round(9.464);
```

![](images/DQL%20数据查询语言/file-20250518200904.png)

round(x,y) 四舍五入，保留y位小数

```sql
select round(9.765, 2);
```

![](images/DQL%20数据查询语言/file-20250518200958.png)

3、truncate(x, y) 舍去

```sql
-- 保留两位小数，剩下的全部舍去
select truncate(9.999, 2);
```

![](images/DQL%20数据查询语言/file-20250518201113.png)

4、ceil 和 floor

ceil 函数：返回大于或等于数值 x 的最小整数

```sql
select ceil(5.3);
```

![](images/DQL%20数据查询语言/file-20250518201223.png)

floor 函数：返回小于或等于数值x的最大整数

```sql
select floor(5.3);
```

5、空处理

ifnull(x, y)：空处理函数，当 x 为 NULL 时，将其替换为 y

```sql
SELECT 
  ename,
  sal,
  comm,
  (sal + IFNULL(comm, 0)) * 12 AS annual_salary
FROM emp;
```

>在 sql 中所有与 null 的运算结果都是 null，为了避免这种情况就可以把 y 设为 0

****
## 5.3 日期和时间相关函数

1、获取当前日期和时间

now()：获取的是执行 select 语句的时刻

now()：获取的是执行 select 语句的时刻

```sql
select sysdate(), sleep(2), now();
```

![](images/DQL%20数据查询语言/file-20250518204849.png)

2、获取当前日期

```sql
select curtime();

select current_time();

select current_time;
```

![](images/DQL%20数据查询语言/file-20250518205036.png)

3、获取单独的年、月、日、时、分、秒

```sql
select year(now());
select month(now());
select day(now());
select hour(now());
select minute(now());
select second(now());
```

>这些函数在使用的时候，需要传递一个日期参数给它，它可以获取到你给定的这个日期相关的年、月、日、时、分、秒的信息

```sql
select date(now()) -- 一次性提取一个给定日期的“年月日”部分
select time(now()) -- 一次性提取一个给定日期的“时分秒”部分
```

4、date_add 函数

>给指定的日期添加间隔的时间，从而得到一个新的日期

```sql
-- date_add(日期, interval expr 单位)
select date_add('2021-07-08', interval 2 month);
```

![](images/DQL%20数据查询语言/file-20250518210131.png)

5、date_format 日期格式化函数

>将日期转换成具有某种格式的日期字符串，通常用在查询操作当中（date类型转换成char类型）

- 第一个参数：这个参数就是即将要被格式化的日期，类型是 date 类型
- 第二个参数：指定要格式化的格式字符串。
  - %Y：四位年份
  - %y：两位年份
  - %m：月份（1..12）
  - %d：日（1..30）
  - %H：小时（0..23）
  - %i：分（0..59）
  - %s：秒（0..59）

```sql
-- 获取当前系统时间，让其以这个格式展示：2000-10-11 20:15:30
select date_format(now(), '%Y-%m-%d %H:%i:%s');
```

![](images/DQL%20数据查询语言/file-20250518210607.png)

6、dayofweek、dayofmonth、dayofyear 函数

```sql
select dayofweek(now()); -- 当前是一周的第几天（周日为第一天）
select dayofmonth(now());
select dayofyear(now());
```

7、last_day 函数

```sql
select last_day(now()); -- 获取给定日期所在月的最后一天的日期
```

8、datediff 函数

```sql
select datediff('2019-1-12', '2016-3-31'); -- 计算两个日期之间所差天数(前面的减后面的，时分秒不算)
```

9、timediff函数

```sql
select timediff('2019-1-1 23:12:47', '2016-3-23 12:30:30'); -- 计算两个日期所差时间（计算时分秒）
```

![](images/DQL%20数据查询语言/file-20250518212144.png)

****
## 5.4 if 函数

```sql
SELECT IF(500<1000, "YES", "NO"); -- 如果条件为 TRUE 则返回 YES，如果条件为 FALSE 则返回 NO，类似于三目运算符
```

****
## 5.5 case 表达式

```sql
CASE field
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    ELSE default_result
END

-- 类似于
switch(field) {
  case value1: return result1;
  case value2: return result2;
  default: return default_result;
}
```

- case：开始条件判断语句
- when：条件
- then：满足条件时返回的值
- else：所有条件都不满足时返回的值
- end：结束条件判断语句

```sql
select ename,job,
	case job
	when 'MANAGER' then sal*1.1
	when 'SALESMAN' then sal*1.2
	else sal
	end 
	as sal
	from emp;
-- manager 的薪水上涨 10%，salesman 的薪水上涨 20%，其余的不变，
```

****
## 5.6 cast 函数

>cast 函数用于将值从一种数据类型转换为表达式中指定的另一种数据类型

```sql
-- cast(值 as 数据类型)
select cast('1999-1-12 12:29:45' as date); -- 将字符串转换为 date 类型
```

![](images/DQL%20数据查询语言/file-20250518215004.png)

****
## 5.7 加密函数

>md5 函数，可以将给定的字符串经过 md5 算法进行加密处理，字符串经过加密之后会生成一个固定长度 32 位的字符串，md5 加密之后的密文通常是不能解密的

```sql
select md5('123');
```

![](images/DQL%20数据查询语言/file-20250518215200.png)

****
# 6. 分组函数

>分组函数的执行原则：先分组，然后对每一组数据执行分组函数，如果没有分组语句 group by 的话，整张表的数据自成一组，如果使用了 where 关键字就一定要使用 group by 才能使用以下分组函数

1、max

```sql
select max(sal) from emp; -- 找出员工的最高薪资
```

2、min

```sql
select min(sal) from emp;
```

3、avg

```sql
select avg(sal) from emp; -- 计算平均值
```

4、sum

```sql
select sum(sal) from emp;
```

5、count

```sql
select count(ename) from emp;
select count(*) from emp;
select count(1) from emp;
```

>`count(*)` 和 `count(1) `的效果一样，统计该组中总记录行数，而 `count(ename)` 统计的是这个ename 字段中不为 NULL 的个数总和

```sql
select count(distinct job) from emp; -- 统计岗位数量
```

****
# 7. 分组查询

## 7.1 group by

