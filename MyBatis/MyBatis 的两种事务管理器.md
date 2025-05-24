
在 mybatis-config.xml 文件中，可以通过以下配置进行 mybatis 的事务管理：

```xml
<transactionManager type="JDBC"/>      <!-- 默认使用 -->
<transactionManager type="MANAGED"/>   <!-- Spring 环境中常用 -->
```

# 1. JDBC 事务管理器

>mybatis 框架自己管理事务，采用原生的 JDBC 代码去管理事务

```java
SqlSession sqlSession = sqlSessionFactory.openSession(); 
// 底层使用的是 conn.setAutoCommit(false); 开启事务
...业务处理...
sqlsession.commit(); // 底层使用的是 conn.commit();手动提交事务
```

openSession() 方法默认的 autoCommit 是 false，也就是默认要自动提交

![](images/MyBatis%20的两种事务管理器/file-20250524191543.png)

这里进入事务管理工厂，通过它来创建事务管理器

![](images/MyBatis%20的两种事务管理器/file-20250524191720.png)

调用 transactionFactory.newTransaction() 来创建具体的事务管理器，进入后可以看到它创建的就是 JDBC 的事务管理器

![](images/MyBatis%20的两种事务管理器/file-20250524191924.png)

进入 JDBC 具体的构造方法后可以发现此时的 autoCommit = false

![](images/MyBatis%20的两种事务管理器/file-20250524192045.png)

然后它会一步一步地走到 openConnection() 方法

![](images/MyBatis%20的两种事务管理器/file-20250524192201.png)

然后进入 setDesiredAutoCommit() 方法

![](images/MyBatis%20的两种事务管理器/file-20250524192328.png)

当执行到 connection.setAutoCommit(desiredAutoCommit) 时，因为传入的 desiredAutoCommit 为 false，所以就可以看作是开启事务









****
# 2. MANAGED 事务管理器

>mybatis 不再负责事务的管理，事务管理交给其他容器负责，对于当前单纯只有 mybatis 的代码来说，如果配置了 MANAGED 那么事务就是没人管理的，即事务没有打开，那也就不用提交事务



