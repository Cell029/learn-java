
# 1. SQL 性能检测工具

## 1.1 查看 MySQL 服务器级别的运行状态指标

>以下指标记录了自服务器启动以来累计的操作次数，而不是当前连接或当前语句的表现

常用的 `Com_` 系列状态变量说明 ：

|命令|含义|
|---|---|
|`Com_select`|执行了多少次 SELECT 查询|
|`Com_insert`|执行了多少次 INSERT|
|`Com_update`|执行了多少次 UPDATE|
|`Com_delete`|执行了多少次 DELETE|
|`Com_commit`|提交事务的次数|
|`Com_rollback`|回滚事务的次数|
|`Com_begin`|显式开启事务的次数|
|`Com_create_table`|创建表的次数|
|`Com_alter_table`|修改表结构的次数|

```sql
SHOW GLOBAL STATUS LIKE 'Com_SQL语句';
```

****
## 1. 2 如何利用这些值做性能分析

#### 1. 查询频率过高

```sql
SHOW GLOBAL STATUS LIKE 'Com_select';
```

如果 `Com_select` 值非常高，而应用响应慢，可能说明：

- 查询语句效率低（如无索引，进行了全表扫描）
- 某些代码重复发送了无用 SQL
- 某些前端组件轮询接口频率过高

****
#### 2. 写操作频率异常

```sql
SHOW GLOBAL STATUS LIKE 'Com_insert';
SHOW GLOBAL STATUS LIKE 'Com_update';
SHOW GLOBAL STATUS LIKE 'Com_delete';
```

如果 INSERT 很高但 SELECT 很少，说明系统是一个写多读少的场景（比如日志采集、监控系统）；而 UPDATE 和 DELETE 频繁，可能是临时表或大量修改的业务场景，也可能存在不合理的更新逻辑

>可用减少 UPDATE/DELETE 的频率，比如使用逻辑删除；将频繁变动数据与静态数据拆分表

****
#### 3. 写入和事务分析

```sql
SHOW GLOBAL STATUS LIKE 'Com_commit';
SHOW GLOBAL STATUS LIKE 'Com_rollback';
```

可以观察是否存在频繁的事务回滚；判断写入压力是否因为事务导致性能瓶颈

****
# 2. 慢查询日志

>慢查询日志是 MySQL 提供的一种日志机制，用于记录执行时间较长（慢）的 SQL 查询语句，比如设定阈值为 3 秒，那么任何 SQL 执行超过 3 秒都会被记录下来，然后有针对性地进行优化，从而提高系统的整体效率

****
## 2.1 慢查询日志的开启

>默认情况下 MySQL 数据库不开启慢查询日志，因为多多少少会带来一定性能的影响

### 1. 查看慢查询日志是否开启

```sql
SHOW VARIABLES LIKE '%slow_query_log';
```

![](images/MySQL%20优化/file-20250523163023.png)

****
### 2. 开启慢查询日志

```sql
SET GLOBAL slow_query_log = ON;
```

也可以在 `my.cnf`（或 `my.ini`）配置文件中永久启用（修改配置文件后需重启 MySQL 才生效）：

```sql
-- 找到 [mysqld] 区域
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 1      -- 默认10秒，建议设为1或更低
log_queries_not_using_indexes = 1  -- 是否记录未使用索引的SQL
```

- `slow_query_log`：是否启用慢查询日志，1 表示开启
- `slow_query_log_file`：指定慢查询日志保存路径
- `long_query_time`：查询耗时超过该值（秒）就记录，建议设为 1 或更低
- `log_queries_not_using_indexes`：是否记录未使用索引的 SQL 查询

****
### 3. 修改慢查询阈值

```sql
SET GLOBAL long_query_time = 1;
```

>改变的是全局默认值，之后任何新的连接（包括重新连 MySQL）都会使用这个新值，但已经打开的连接仍然使用它们各自的 `SESSION` 值（不会立刻改变），重启 MySQL 后会恢复为配置文件中的默认值，除非写进 `my.cnf` / `my.ini` 配置文件

```sql
SET SESSION long_query_time = 1;
```

>这个参数的值只在当前登录的这条 MySQL 会话中生效，一旦断开连接，设置就失效，新开的连接默认仍是 `10.000000` 秒（或配置文件中的值）

如果希望当前连接立即生效 + 后续连接也生效，就可以同时执行以上两个

****
## 2.2 慢查询日志的位置

### 1. 查看慢查询日志的位置

```sql
SHOW VARIABLES LIKE 'slow_query_log_file';
```

****
### 2. 修改慢查询日志位置

```sql
set global slow_query_log_file = '...-slow.log';
```

****
## 2.3 慢 SQL 的查看

### 1. 查看慢查询日志内容

执行了一条模拟

```sql
E:\mysqlEnvironment\mysql-8.4.5-winx64\bin\mysqld, Version: 8.4.5 (MySQL Community Server - GPL). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
# Time: 2025-05-23T09:23:56.446824Z
# User@Host: root[root] @ localhost [::1]  Id:     9
# Query_time: 2.007068  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use demo_01;
SET timestamp=1747992234;
select sleep(2);
```

- `Time`：执行时间
- `User@Host`：发起请求的用户和 IP
- `Query_time`：执行 SQL 花费的总时间（秒）
- `Lock_time`：等待锁的时间
- `Rows_sent`：返回的记录数
- `Rows_examined`：扫描的记录数
- `SET timestamp=...`：与 Time 一致的秒数形式，用于重现执行时间

>通过 `Query_time` 和 `Rows_examined` 可以快速判断 SQL 是否有性能问题

```sql
-- 查询当前系统中有多少条慢查询记录
SHOW GLOBAL STATUS LIKE '%Slow_queries%';
```

![](images/MySQL%20优化/file-20250523174236.png)

****
### 2. SHOW PROFILES 查看 SQL 耗时

1、查看当前数据库是否支持 profile 操作：

```sql
select @@have_profiling;
```

2、查看 profiling 开关是否打开（默认关闭）：

```sql
select @@profiling; -- 0 关闭，1 打开
```

3、将 profiling 开关打开：

```sql
set profiling = 1;
```

4、可用执行多条 DQL 语句，然后使用 `show profiles` 查看当前数据库中执行过的每个SELECT语句的耗时情况：

```sql
select empno,ename from emp;
select empno,ename from emp where empno=7369;
select count(*) from emp;
show profiles;
```

![](images/MySQL%20优化/file-20250523175153.png)

- `Query_ID`: 执行顺序编号，越大表示越晚执行
- `Duration`: 总耗时（秒）
- `Query`: 执行的 SQL 内容

5、查看某一条 SQL 的详细执行阶段：

```sql
SHOW PROFILE FOR QUERY Query_ID;
```

![](images/MySQL%20优化/file-20250523175648.png)

- `starting`：启动 SQL 解析器
- `Executing hook on tt`：执行表的特殊钩子函数（通常与存储引擎相关）
- `checking permissions`：检查用户权限
- `Opening tables`：打开表文件
- `init`：初始化临时表或缓存（如子查询优化）
- `System lock`：等待表级锁或行级锁
- `optimizing`：选择最佳执行路径（使用索引、连接顺序等）
- `statistics`：收集表统计信息，用于优化器决策
- `preparing`：预处理阶段（生成执行计划）
- `executing`：实际开始执行 SQL（如取数据）
- `Sending data`：发送结果数据给客户端
- `end`：执行结束准备清理资源
- `query end`：查询正式结束（等待存储引擎确认完成）
- `waiting for handler commit`：线程正在等待存储引擎完成事务的提交操作
- `closing tables`：关闭已打开的表
- `freeing items`：释放内存缓存（如临时表、排序缓冲区）
- `cleaning up`：最终清理（重置线程状态）

****
# 3. EXPLAIN 执行计划

>使用 `EXPLAIN` 关键字可以模拟优化器执行 SQL 语句，分析查询语句或结构的性能瓶颈。 在 `SELECT` 语句之前增加 `EXPLAIN` 关键字，MySQL 会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行这条 SQL（如果 from 中包含子查询，仍会执行该子查询，将结果放入临时表中）

****
## 3.1 两个变种

### 1. EXPLAIN FORMAT=JSON

>返回更结构化、更详细的执行计划信息（如索引使用、过滤条件、连接类型、排序方式等）。输出是 JSON 格式，适合工具自动化解析或深入分析优化器行为

```sql
EXPLAIN FORMAT=JSON SELECT * FROM film WHERE id = 1;
```

```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.00"
    },
    "table": {
      "table_name": "film",
      "access_type": "const",
      "possible_keys": [
        "PRIMARY"
      ],
      "key": "PRIMARY",
      "used_key_parts": [
        "id"
      ],
      "key_length": "4",
      "ref": [
        "const"
      ],
      "rows_examined_per_scan": 1,
      "rows_produced_per_join": 1,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "0.00",
        "eval_cost": "0.10",
        "prefix_cost": "0.00",
        "data_read_per_join": "72"
      },
      "used_columns": [
        "id",
        "name"
      ]
    }
  }
}
```

常见字段：

- `select_id`：查询块的 ID，子查询会有多个
- `table_name`：查询的表名
- `access_type`：表访问类型，如 `const`（常量级别访问）、`ref`（基于索引的非唯一等值查找）、`ALL`（全表扫描）
- `possible_keys`：查询可用的索引
- `key`：实际使用的索引
- `rows`：预计扫描的行数
- `filtered`：过滤后保留的比例（百分比）

****
### 2. EXPLAIN ANALYZE

>实际执行 SQL 语句（区别于普通 EXPLAIN 只是估算）并显示真实的执行时间、扫描行数、每一步的耗时，主要用于深入性能分析与调优，比普通 `EXPLAIN` 更真实

```sql
EXPLAIN ANALYZE SELECT * FROM film WHERE id = 1;
```

```sql
-> Rows fetched before execution  (cost=0..0 rows=1) (actual time=0.0011..0.0012 rows=1 loops=1)
```

关键字段：

- `Rows fetched before execution`：MySQL 在真正进入执行计划前就能确定结果，因为语句中是 `WHERE id = 1`，而 `id` 是主键，所以它知道只要查一行、直接定位，无需真正执行复杂计划
- `cost=0..0 rows=1`：优化器的估算成本和行数
- `actual time=0.0011..0.0012`：实际执行开始和结束时间（单位：秒）
- `rows=1 loops=1`:实际返回的行数和执行次数

****
## 3.2 EXPLAIN 的列

### 1. id：查询块标识

>每个 `SELECT` 子句的编号，值越大，执行优先级越高，如果是子查询或联合查询，就会有多个 ID

****
### 2. select_type：查询类型

- `SIMPLE`：简单查询，不包含子查询或 UNION
- `PRIMARY`：最外层的 SELECT
- `SUBQUERY`：子查询
- `DERIVED`：派生表（FROM 子句中的子查询）
- `UNION`：UNION 中的第二个及后续查询
- `UNION RESULT`：UNION 合并结果集
- `DEPENDENT SUBQUERY`：依赖外层的子查询

****
### 3. table：访问的表

>显示当前执行计划中正在访问的表或别名，如果是临时表则显示 `derived` 或 `union result`

****
### 4. partitions：分区信息（如无分区则为空）

>指示命中了哪些分区（对分区表才有用）

****
### 5. type：连接类型（访问类型）

- `system`：表仅有一行（等价于 const）
- `const`：使用主键或唯一索引进行等值查询
- `eq_ref`：对于驱动表（前一个表）中的每一行，MySQL 会使用某个索引，到被驱动表中精确地查找一行数据进行匹配
- `ref`：非唯一索引或前缀索引，返回多行匹配项
- `range`：使用索引范围查找，如 `BETWEEN`、`<`、`>`
- `index`：全索引扫描（比全表扫描好）
- `ALL`：全表扫描（最差）

****
### 6. 总结

| 列名              | 含义                           |
| --------------- | ---------------------------- |
| `id`            | 查询中每个 SELECT 子句或 UNION 的唯一标识 |
| `select_type`   | SELECT 的类型，如简单查询、子查询、派生表等    |
| `table`         | 当前正在访问的表名                    |
| `partitions`    | 显示命中查询的分区（如果表使用了分区）          |
| `type`          | 连接类型（访问类型），表示表访问方式的效率        |
| `possible_keys` | 查询可能用到的索引（优化器预估）             |
| `key`           | 实际使用的索引                      |
| `key_len`       | 使用的索引长度（字节）                  |
| `ref`           | 哪些列或常量与 key 进行了匹配            |
| `rows`          | 预估需要读取的行数                    |
| `filtered`      | 表示行过滤率（百分比）                  |
| `Extra`         | 附加信息（如是否使用临时表、文件排序等）         |

```sql
EXPLAIN
SELECT a.name, f.name AS film_name
FROM actor a
JOIN film_actor fa ON a.id = fa.actor_id
JOIN film f ON fa.film_id = f.id
WHERE f.release_year BETWEEN 2000 AND 2010
  AND a.name LIKE 'A%'
ORDER BY f.release_year DESC
LIMIT 10;
```

| id  | select_type | table | partitions | type   | possible_keys     | key               | key_len | ref                | rows | filtered | Extra                                                   |
| --- | ----------- | ----- | ---------- | ------ | ----------------- | ----------------- | ------- | ------------------ | ---- | -------- | ------------------------------------------------------- |
| 1   | SIMPLE      | a     | NULL       | ALL    | PRIMARY           | NULL              | NULL    | NULL               | 5    | 20.00    | Using where; Using temporary; Using filesort            |
| 1   | SIMPLE      | fa    | NULL       | index  | idx_film_actor_id | idx_film_actor_id | 8       | NULL               | 7    | 14.29    | Using where; Using index; Using join buffer (hash join) |
| 1   | SIMPLE      | f     | NULL       | eq_ref | PRIMARY           | PRIMARY           | 4       | demo_01.fa.film_id | 1    | 11.11    | Using where                                             |

- **id = 1，select_type = SIMPLE**  

>说明这是主查询（非子查询）

- **table = a**  
>表示当前处理的是 `actor` 表

- **partitions = NULL**  

>没用分区

- **type = ALL**  

>全表扫描，这是比较低效的连接方式，说明索引没有被用在这个表上，可能是因为 `a.name LIKE 'A%'` 这类条件不能有效用索引（字符串前缀匹配可以用索引，但需要确认字段和索引情况）

- **possible_keys = PRIMARY**  

>优化器认为有主键索引可用，但没选用

- **key = NULL**  

>实际没使用任何索引

- **key_len = NULL** 

>无索引使用

- **ref = NULL**  

>无引用字段

- **rows = 5**  

>估计需要扫描5行

- **filtered = 20.00**  

>过滤效率20%，表示大概20%的行会通过 `WHERE` 条件过滤

- **Extra = Using where; Using temporary; Using filesort**
>`Using where`：有 `WHERE` 条件过滤（你例子里的 `a.name LIKE 'A%'`）
>
>`Using temporary`：因为后面有 `ORDER BY`，MySQL 需要临时表来存储排序结果
>
>`Using filesort`：排序操作不能使用索引，需要额外的排序步骤

****
# 4. 索引失效问题

先给表添加两个索引：

```sql
ALTER TABLE users ADD PRIMARY KEY p_user_id(user_id); -- 主键索引
ALTER TABLE users ADD KEY unite_index(user_name,user_sex,password); -- 组合索引
```

### 1. 查询中带有OR会导致索引失效

```sql
EXPLAIN SELECT * FROM `zz_users` WHERE user_id = 1 OR user_name = "熊猫";
```




