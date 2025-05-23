
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







