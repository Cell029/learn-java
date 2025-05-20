
# 1. 创建用户

创建一个用户名为java1，密码设置为123的本地用户：

```sql
create user 'java1'@'localhost' identified by '123';
```

>该用户只拥有本地登录权限，不能创建

![](images/DBA%20命令/file-20250520205033.png)

创建一个用户名为java2，密码设置为123的外网用户：

```sql
create user 'java2'@'%' identified by '123';
```

****
# 2. 给用户授权


