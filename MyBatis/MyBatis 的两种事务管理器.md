
在 mybatis-config.xml 文件中，可以通过以下配置进行 mybatis 的事务管理：

```xml
<transactionManager type="JDBC"/>      <!-- 默认使用 -->
<transactionManager type="MANAGED"/>   <!-- Spring 环境中常用 -->
```

![](images/MyBatis%20的两种事务管理器/file-20250524194702.png)

****
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

当执行到 connection.setAutoCommit(desiredAutoCommit) 时，因为传入的 desiredAutoCommit 为 false，即 MyBatis 在默认情况下是主动将连接设为非自动提交，也就是可以看作开启事务，但必须在操作后手动 commit

****

如果使用了：

```java
SqlSession sqlSession = sqlSessionFactory.openSession(true); 
// 底层使用的是 conn.setAutoCommit(false); 开启事务
...业务处理...
sqlsession.commit(); // 底层使用的是 conn.commit();手动提交事务
```

那在这个源码的部分，if (!this.skipSetAutoCommitOnClose && !this.connection.getAutoCommit()) 就会因为 true != true 而进不去 connection.setAutoCommit() 方法，那么 JDBC 底层就会按照默认的情况，开启自动提交，此时就不需要再手动使用 sqlsession.commit()

![](images/MyBatis%20的两种事务管理器/file-20250524193307.png)

****
# 2. MANAGED 事务管理器

>mybatis 不再负责事务的管理，事务管理交给其他容器负责，对于当前单纯只有 mybatis 的代码来说，如果配置了 MANAGED 那么事务就是没人管理的，即事务没有打开，那也就不用提交事务

也就是说 `ManagedTransaction` 根本不会控制事务的提交和回滚，它的实现是空操作，所有事务行为都依赖外部容器（如 Spring、JEE 容器等），如果没有外部容器接管，它就默认保持数据库连接的默认行为，通常是 `autoCommit = true`

****
# 3. 总结

>JDBC 中，如果 `autoCommit = true`，则每条 SQL 都是一个隐式事务，执行完立即提交，就不存在可控制的事务边界，此时的 `rollback()` 调用就会变得无效，当业务逻辑中包含多条写操作（update、insert、delete）组合为一个原子操作时，无法保证这些语句的原子性，一旦失败就不能回滚，这是十分不安全的

****