
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
- `ALL`：全表扫描（最差，代表索引失效）

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
ALTER TABLE users ADD KEY unite_index(user_name,password); -- 组合索引
```

```sql
SHOW INDEX FROM users;
```

![](images/MySQL%20优化/file-20250523205430.png)

****
### 1. 不满足最左前缀

>最左前缀原则是指：在使用联合索引（复合索引）时，查询条件必须从索引定义的最左边字段开始按顺序使用才能有效命中索引，否则就会导致索引失效

```sql
EXPLAIN SELECT * FROM users WHERE password = "1234";
```

![](images/MySQL%20优化/file-20250523210018.png)

从结果中看到 `type = ALL`，索引完全失效，使用了全表扫描。联合索引底层是按组合键排序的，如果查询时不指定最左字段，MySQL 就无法快速定位第一条数据的位置，也就无法利用索引

```sql
EXPLAIN SELECT * FROM users WHERE user_name = "熊猫";
```

![](images/MySQL%20优化/file-20250523210213.png)

使用了 `user_name` 后就可以看到 MySQL 底层进行了优化，使用了组合索引中的前缀索引，没有导致索引失效

****
### 2. 范围查询之后

>如果对其中一个字段使用范围查询会导致部分索引失效

```sql
EXPLAIN SELECT * FROM users WHERE user_name > '熊' AND password = '1234';
```

![](images/MySQL%20优化/file-20250523211324.png)

可以看到 `type = range` ，表示使用了索引的范围扫描方式，这是比全表扫描更优的一种访问类型，但 `password` 字段未使用索引，是在回表之后再判断的，因为范围查询会导致后续字段无法使用索引

为什么范围查询后面的字段索引失效：

>当定位到某个确切的键值时，B+ 树可以精准定位到该键下对应的所有数据，然后继续利用剩余字段判断，但如果是范围查询那 MySQL 就只能定位一个区间，可区间内可能有很多键值，在这个范围内索引树结构已经无法再有效细分（此时可能定位到两个叶子节点区间），所以必须扫描范围内所有符合进行范围判断的字段的行，拿到主键后再回表过滤剩余字段

****
### 3. 查询中带有 OR

>只要一个 `OR` 条件中的任意一个子条件无法使用索引，整个 SQL 查询将不会使用索引（即：全表扫描），无论其他子条件是否可以使用索引

```sql
EXPLAIN SELECT * FROM users WHERE user_id = 1 OR user_sex = "男";
```

![](images/MySQL%20优化/file-20250523205515.png)

从结果中看到 `type = ALL`，索引完全失效，使用了全表扫描，虽然用到了主键，但是后面的 `user_sex` 并不是索引

****
### 4. 索引字段做运算或参与函数

>索引字段做运算或使用函数时会导致 MySQL 无法使用索引的有序性进行快速定位，因为被处理后的表达式不再直接对应索引中存储的结构，索引结构就错位了，必须扫描全表来判断每一行是否满足条件

```sql
EXPLAIN SELECT * FROM users WHERE user_id + 1 = 2;
```

```sql
EXPLAIN SELECT * FROM users WHERE LEFT(user_name, 2) = '熊猫';
```

![](images/MySQL%20优化/file-20250523213029.png)

****
### 5. MySQL 进行隐式类型转换

>如果对字符串字段查询时没有加单引号，MySQL 会进行隐式类型转换，这将导致无法使用索引，进而降低查询性能

```sql
EXPLAIN SELECT * FROM users WHERE password = 1234;
```

![](images/MySQL%20优化/file-20250523213701.png)

因为 `password` 是 `varchar` 类型的，查询时没使用 `""` ，MySQL 在执行查询时有个查询优化器，它会尝试选择最佳的执行计划，当写错数据类型（把字符串常量写成没有引号的裸字），优化器会认为需要对字段进行函数转换，就会变成上一个类型，导致进行全表扫描

****
### 6. SELECT *

>使用 `SELECT *` 本身不会直接导致索引失效，但在某些情况下它可能间接导致无法使用覆盖索引，从而影响性能

```sql
EXPLAIN SELECT user_name, password FROM users WHERE user_name = '熊猫';
```

![](images/MySQL%20优化/file-20250523214511.png)

而 `SELECT *` 会导致 MySQL 不知道要哪些字段，就会默认要所有字段（包括主键、未被索引的字段等），如果查询的字段有些不在索引里，MySQL 必须回表读取完整数据行

>但索引并没有完全失效，因为 `WHERE` 中使用了索引中的字段，只不过会多个回表的过程

```sql
EXPLAIN SELECT * FROM users WHERE user_name = '熊猫';
```

****
### 7. 模糊查询导致失效

1、`LIKE '%xxx'`（前面有通配符）

>`%` 在前面表示匹配任意开头，MySQL 无法利用索引做范围查找，因为 B+ 树索引是按顺序存储的，所以只能从开头定位，然后就变成了全表扫描，`LIKE '%xxx%'`（两边都有通配符）、`LIKE '_xxx'` 同理

```sql
EXPLAIN SELECT user_name, password FROM users WHERE user_name like '%猫';
```

![](images/MySQL%20优化/file-20250523215728.png)

严格来说，索引没有完全失效，因为 MySQL 使用了索引覆盖扫描（`type = index`），没有回表，减少了读取数据行的开销，但它肯定扫描了整个索引（类似全表扫描），性能依旧较低

2、`LIKE 'xxx%'`（右侧通配符）

>这是最常见的前缀匹配，MySQL 可以使用索引做范围查找，并不会导致索引失效

****
### 8. MySQL 有时选择全表扫描而不用索引

>使用索引虽然可以减少扫描的行数，但会有额外的回表成本（访问数据行，尤其是非覆盖索引时），如果预估返回的数据行数比较多（比如超过总表的某个比例，比如30%），MySQL 则认为用索引反而更慢，于是直接选择全表扫描，减少回表的开销

****
### 9. IS NULL 和 IS NOT NULL

>`IS NULL` 查询是可以走索引的，因为索引中会存储字段值为 NULL 的条目，MySQL 的 B+ 树索引中会把 `NULL` 当成一个特殊的键值来存储； `IS NOT NULL` 也可以利用索引，但效果不一定好，所有非 NULL 的值都在索引里，由于匹配范围广（可能绝大部分都是非 NULL），MySQL 优化器可能选择不使用索引，直接做全表扫描更快

****
### 10. 反向操作

>对索引列使用 `NOT IN`, `!=` 等条件时，查询通常不会走索引，或走索引效率非常低，使用 `NOT LIKE 'xxx%'` 也会导致索引失效，因为无法做范围扫描

```sql
EXPLAIN SELECT * FROM users WHERE user_name NOT LIKE '熊%';
```

![](images/MySQL%20优化/file-20250523221506.png)

```sql
EXPLAIN SELECT * FROM users WHERE user_name <> '熊猫';
```

![](images/MySQL%20优化/file-20250523221545.png)

`range` 类型不代表索引失效，而是部分利用索引（范围扫描），但效率可能不如等值查询

****
# 5. 索引优化（正确使用）

>实际上就是根据前面给出的索引失效情况，尽量让自己编写的 `SQL` 不会导致索引失效即可，写出来的 `SQL` 能走索引查询，那就能在很大程度上提升数据检索的效率

****
## 5.1 索引覆盖

>由于表中只能存在一个聚簇索引，一般都为主键索引，而建立的其他索引都为辅助索引，包括联合索引也不例外，所以最终索引节点上存储的都是指向主键索引的值

```sql
EXPLAIN SELECT * FROM users WHERE user_name = "竹子" AND user_sex = "男";
```

虽然这条 `SQL` 会走联合索引查询，但是基于联合索引查询出来的值仅是一个指向主键索引的 `ID`，然后会拿着这个 `ID` 再去主键索引中查一遍，但将 `SQL` 更改为查询索引中拥有的列后，就不会发生回表现象：[2. 覆盖索引](索引.md#2.%20覆盖索引)

****
## 5.2 索引下推

>当执行带有 `WHERE` 条件的查询时，MySQL 在使用索引扫描时会先根据索引条件定位相关索引记录：在传统执行方式下，MySQL 会先通过索引找到满足索引字段条件的所有行（获取主键或行指针），然后再回表根据查询的非索引字段过滤剩余条件；索引下推则是把部分过滤条件直接下推到索引层面，在扫描索引时就先过滤掉不符合条件的记录，减少回表的行数从而提高查询效率

```sql
SELECT * FROM users WHERE user_name LIKE "竹%" AND user_sex="男";
```

由于使用的是模糊查询，但 `%` 在结尾，因此可以使用 `竹` 这个字作为条件在联合索引中查询，整个查询过程如下：

- 1、利用联合索引中的 `user_name` 字段找出「竹子、子竹、竹竹」三个索引节点；
- 2、返回索引节点存储的值「`2、3、10`」给 `Server` 层，然后去逐一做回表扫描；
- 3、在 `Server` 层中根据 `user_sex="男"` 这个条件逐条判断，最终筛选到「竹子」这条数据。

理论上利用索引下推后的过程如下：

- 1、利用联合索引中的 `user_name` 字段找出「竹子、子竹、竹竹」三个索引节点；
- 2、根据` user_sex="男"` 这个条件在索引节点中逐个判断，从而得到「竹子」这个节点；
- 3、最终将「竹子」这个节点对应的「`2`」返回给 `Server` 层，然后聚簇索引中回表拿数据。

相较于没有索引下推之前，原本需要做「`2、3、10`」三次回表查询，但在拥有索引下推之后，仅需做「`2`」一次回表查询

>索引下推（Index Condition Pushdown，简称ICP） 是 MySQL 5.6 版本之后引入并默认开启的一个优化功能，也可以通过以下命令管理：

```sql
-- 查看当前设置
SHOW VARIABLES LIKE 'optimizer_switch';

-- 关闭ICP
SET optimizer_switch='index_condition_pushdown=off';

-- 开启ICP（默认是on）
SET optimizer_switch='index_condition_pushdown=on';
```

****
## 5.3 MRR(Multi-Range Read) 机制

>多范围读取（扫描），它的主要目标是减少磁盘随机读取次数，优化从磁盘中读取多个不连续数据页的效率

```sql
SELECT * FROM score WHERE grade BETWEEN 0 AND 59;
```

流程如下：

- 1、先在成绩字段的索引上找到 0 分的节点，然后拿着 `ID` 去回表得到成绩零分的学生信息；
- 2、再次回到成绩索引，继续找到所有 1 分的节点，继续回表得到 1 分的学生信息；
- 3、再次回到成绩索引，继续找到所有 2 分的节点......
- 4、不断重复这个过程，直到将 0~59 分的所有学生信息全部拿到为止。

那此时假设此时成绩 0~5 分的表数据，位于磁盘空间的 `page_01` 页上，而成绩为 5~10 分的数据位于磁盘空间的 `page_02` 页上，成绩为 10~15 分的数据，又位于磁盘空间的 `page_01` 页上。此时回表查询时就会导致在 `page_01`、`page_02` 两页空间上来回切换，但 0~5、10~15 分的数据完全可以合并，然后读一次 `page_01` 就可以了，既能减少 IO 次数，同时还避免了离散 IO

>先扫描索引，根据查询条件定位符合条件的索引条目（一般是主键或聚簇索引的行指针），但不立即回表读取数据行，接着将对应数据行所在的数据页号全部收集起来，存在一个缓冲区，并对数据页号排序（通常是页号的顺序），以优化后续的磁盘访问，最后按顺序访问这些数据页，一次读取一页，根据已读取的数据页返回符合条件的数据，减少磁盘 IO 次数。`MySQL5.6` 及以后的版本是默认开启的：

```sql
-- 查看当前设置
SHOW VARIABLES LIKE 'innodb_use_mrr';

-- 关闭MRR
SET GLOBAL innodb_use_mrr = OFF;

-- 开启MRR
SET GLOBAL innodb_use_mrr = ON;
```

****
## 5.4 Index Skip Scan 索引跳跃式扫描

>最左前缀匹配原则就是 `SQL` 的查询条件中必须要包含联合索引的第一个字段，这样才能命中联合索引查询，但实际上这条规则也并不是 100% 遵循的。因为在 `MySQL8.x` 版本中加入了一个新的优化机制，也就是索引跳跃式扫描，这种机制使得即使查询条件中没有使用联合索引的第一个字段，也依旧可以使用联合索引，看起来就像跳过了联合索引中的第一个字段一样

假设建立了一个联合索引 `(A、B、C)` ，有如下一条 `SQL`：

```sql
SELECT * FROM tb_xx WHERE B = 'xxx' AND C = 'xxx';
```

优化器会重构 `SQL` ，并不是真正的跳过了第一个字段：

```sql
SELECT * FROM tb_xx WHERE B = 'xxx' AND C = 'xxx'
UNION ALL
SELECT * FROM tb_xx WHERE B = 'xxx' AND C = 'xxx' AND A = "yyy"
......
SELECT * FROM tb_xx WHERE B = 'xxx' AND C = 'xxx' AND A = "zzz";
```

>跳跃扫描中 MySQL 利用索引的有序性，先跳过所有不同的 `A` 值，针对每个 `A` 的值，使用索引查找 `B = ?` 、`C = ?` 的记录

例如：联合索引为 `(customer_id, status)`

```sql
SELECT * FROM orders WHERE status = 'shipped';
```

>MySQL 会在索引中跳过不同的 `customer_id`，然后在每个 `customer_id` 子组中扫描 `status = 'shipped'`，跳过不匹配的条目，这就会比全索引扫描效率高很多，MySQL 8.0.12 后就支持了跳跃扫描（默认开启）：

```sql
-- 查看当前配置
SHOW VARIABLES LIKE 'optimizer_switch';

-- 开启
SET SESSION optimizer_switch = 'skip_scan=on';

-- 关闭
SET SESSION optimizer_switch = 'skip_scan=off';
```

****
# 6. SQL 优化

## 1. 子查询用 EXISTS 代替 IN

>当 IN 的参数是子查询时，数据库首先会执行子查询，然后将结果存储在一张临时的工作表里（内联视图），然后扫描整个视图，但很多情况下这种做法都非常耗费资源。使用EXISTS的话，数据库就不会生成临时的工作表。不过从代码的可读性上来看 IN 要比 EXISTS 好，使用 IN 时的代码看起来更加一目了然，易于理解。因此，如果确信使用 IN 也能快速获取结果，就没有必要非得改成 EXISTS 了。

```sql
SELECT * FROM A WHERE column IN (SELECT column FROM B);

SELECT * FROM A WHERE EXISTS (
    SELECT 1 FROM B WHERE B.column = A.column
);
```

这两种写法在结果上通常是等价的，都是用于筛选符合条件的行，但它们的执行方式不同，导致在性能上会有显著差异，通常使用 EXISTS 会更快点：

>`EXISTS` 子查询一旦找到一条符合条件的记录就终止并返回 TRUE，而 `IN` 会扫描完整个子查询结果集进行比较；`EXISTS` 子查询可通过关联字段的索引快速定位，提高性能，而 `IN` 在很多情况下会生成临时表或不能走索引

****
## 2. 避免排序并添加索引

>在 SQL 语言中，除了 ORDER BY 子句会进行显示排序外，还有很多操作默认也会在底层进行排序，如果排序字段没有添加索引，就会导致查询性能很慢。SQL中会进行排序的代表性的运算有下面这些：

- GROUP BY子句
- ORDER BY子句
- 聚合函数（SUM、COUNT、AVG、MAX、MIN）
- DISTINCT
- 集合运算符（UNION、INTERSECT、EXCEPT）
- 窗口函数（RANK、ROW_NUMBER等）

如上列出的六种运算（除了集合运算符），它们后面跟随或者指定的字段都可以添加索引，这样可以加快排序

****











