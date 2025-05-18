
下载网址：[mysql](https://www.mysql.com/) 

# 1. 配置环境

1、此电脑 -> 属性 -> 高级属性设置 -> 环境变量，打开配置环境变量窗口

![](images/MySQL%20的下载与配置/file-20250518142758.png)

2、点击新建，新建环境变量 MySQL_HOME，变量值是解压之后 bin 文件夹所在的目录

![](images/MySQL%20的下载与配置/file-20250518142905.png)

3、配置 PATH 环境变量，双击 PATH

![](images/MySQL%20的下载与配置/file-20250518142949.png)

![](images/MySQL%20的下载与配置/file-20250518143010.png)

点击新建，输入 %MySQL_HOME%\bin，点击确定

![](images/MySQL%20的下载与配置/file-20250518143202.png)

****
# 2. 初始化

1、以管理员身份运行命令提示符

![](images/MySQL%20的下载与配置/file-20250518143304.png)

2、输入以下命令初始化 data

```
mysqld --initialize --console
```

注意：root@localhost 是初始密码，复制之后保存起来

![](images/MySQL%20的下载与配置/file-20250518143359.png)

3、安装 MySQL 服务

```
mysqld -install
```

删除旧的服务，下载新的

![](images/MySQL%20的下载与配置/file-20250518143918.png)

4、登录 MySQL

先启动 MySQL，然后输入对应的账号密码

![](images/MySQL%20的下载与配置/file-20250518144219.png)

![](images/MySQL%20的下载与配置/file-20250518144300.png)

5、修改密码

![](images/MySQL%20的下载与配置/file-20250518144450.png)

6、登录 MySQL

当 MySQL 服务已经运行时, 我们可以通过 MySQL 自带的客户端工具登录到 MySQL 数据库中, 首先打开命令提示符, 输入以下格式的命名：

```
mysql -h 主机名 -u 用户名 -p
```

- **-h** : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
- **-u** : 登录的用户名;
- **-p** : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

登录本机：

```
mysql -u root -p
```

按回车确认, 如果安装正确且 MySQL 正在运行, 会得到以下响应：

```
Enter password:
```

若密码存在, 输入密码登录, 不存在则直接按回车登录。登录成功后你将会看到 Welcome to the MySQL monitor... 的提示语。

![](images/MySQL%20的下载与配置/file-20250518144737.png)

然后命令提示符会一直以 mysql> 加一个闪烁的光标等待命令的输入, 输入 **exit** 或 **quit** 退出登录
