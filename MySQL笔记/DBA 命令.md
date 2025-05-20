
# 1. 创建用户

创建一个用户名为java1，密码设置为123的本地用户：

```sql
create user 'java1'@'localhost' identified by '123';
```

>该用户只拥有本地登录权限，不能创建

![](images/DBA%20命令/file-20250520205033.png)

创建一个用户名为 java2，密码设置为 123 的外网用户：

```sql
create user 'java2'@'%' identified by '123';
```

****
# 2. 给用户授权

```sql
GRANT 权限列表 ON 库名.表名 TO '用户名'@'主机名或IP地址' [WITH GRANT OPTION];
```

| 元素                | 含义                                                               |
| ----------------- | ---------------------------------------------------------------- |
| 权限列表              | 如：`SELECT`、`INSERT`、`UPDATE` 等，也可使用 `ALL PRIVILEGES` 表示所有权限      |
| 库名.表名             | `*.*` 表示所有库的所有表，`db_name.*` 表示某库的所有表，`db_name.table_name` 表示具体某表 |
| '用户名'@'主机'        | 用户名 + 可登录的主机或 IP；`'%'` 表示任意主机，`'localhost'` 表示本地连接               |
| WITH GRANT OPTION | 被授权用户可以把自己的权限再授权给其他用户（慎用）                                        |

1、给本地用户授权

```sql
-- 允许 java1 用户在本地访问所有数据库的所有表，并拥有增删改查和创建的权限
GRANT SELECT, INSERT, DELETE, UPDATE, CREATE ON *.* TO 'java1'@'localhost';
```

![](images/DBA%20命令/file-20250520210132.png)

2、给远程用户授权

```sql
-- 允许 java1 用户从任意 IP 连接，拥有 demo_01 数据库中所有表的全部权限
GRANT ALL PRIVILEGES ON demo_01.* TO 'java1'@'%';
```

3、授予权限并允许再次授权他人

```sql
-- java2 可以授予其他用户自己所拥有的权限，拥有“授权权”
GRANT SELECT, INSERT, DELETE, UPDATE ON *.* TO 'java2'@'%' WITH GRANT OPTION;
```

4、查看用户权限

```sql
SHOW GRANTS FOR '用户名'@'主机';

SHOW GRANTS FOR 'java1'@'localhost';
SHOW GRANTS FOR 'java2'@'%';
```

![](images/DBA%20命令/file-20250520210506.png)

5、撤销授权

```sql
REVOKE 权限列表 ON 库名.表名 FROM '用户名'@'主机';
```

6、权限刷新

授权或撤销后执行以下命令立即生效：

```sql
FLUSH PRIVILEGES;
```

7、删除用户

```sql
DROP USER '用户名'@'主机';
FLUSH PRIVILEGES;
```

****
# 3. 修改用户的密码

```sql
ALTER USER '用户名'@'主机' IDENTIFIED BY '新密码';

-- 修改本地用户密码
ALTER USER 'java1'@'localhost' IDENTIFIED BY '456';

-- 修改外网用户密码
ALTER USER 'java2'@'%' IDENTIFIED BY '456';

-- 刷新权限
FLUSH PRIVILEGES;
```

****
# 4. 修改用户名

```sql
RENAME USER '旧用户名'@'旧主机' TO '新用户名'@'新主机';

-- 将本地用户 java1 重命名为 java11
RENAME USER 'java1'@'localhost' TO 'java11'@'localhost';

-- 再将 java11 用户从本地变为允许远程登录，用户名同时变更为 java123
RENAME USER 'java11'@'localhost' TO 'java123'@'%';

FLUSH PRIVILEGES;
```

****
# 5. 数据备份

## 5.1 数据导出

1、导出整个数据库

```sql
mysqldump -uroot -p123 --default-character-set=utf8 demo_01 > e:/demo_01.sql
-- 也可以不显示输入密码，执行时再手动输入
```

![](images/DBA%20命令/file-20250520212129.png)

2、导出指定表的数据

```sql
mysqldump -uroot -p123 --default-character-set=utf8 demo_01 emp > e:/demo_01.sql
```

****
## 5.2 数据导入

1、在未登录 MySQL 前导入

```sql
mysql -uroot -p123 --default-character-set=utf8 demo_01 < e:/demo_01.sql
```

2、登录 MySQL 后使用 `source` 命令

```sql
create database demo_01;
use demo_01;
source e:/demo_01.sql
```
