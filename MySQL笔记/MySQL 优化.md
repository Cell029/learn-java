
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
### 1. 