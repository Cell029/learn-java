
## 1. 添加依赖

在 `pom.xml` 中添加依赖：

```xml
<!-- MyBatis 核心依赖 -->  
<dependency>  
  <groupId>org.mybatis</groupId>  
  <artifactId>mybatis</artifactId>  
  <version>3.5.15</version>  
</dependency>  
  
<!-- MySQL 驱动 -->  
<dependency>  
  <groupId>com.mysql</groupId>  
  <artifactId>mysql-connector-j</artifactId>  
  <version>8.2.0</version>  
</dependency>
```

****
## 2. 建表

![](images/MyBatis%20入门程序/file-20250524174450.png)

****
## 3. 配置 MyBatis 核心配置文件

在 `resources` 根目录下新建 `mybatis-config.xml` 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
    <environments default="development">  
        <environment id="development">  
            <transactionManager type="JDBC"/>  
            <dataSource type="POOLED">  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://localhost:3306/demo_01"/>  
                <property name="username" value="root"/>  
                <property name="password" value="123"/>  
            </dataSource>  
        </environment>  
    </environments>  
    <mappers>        
	    <!--执行 XxxMapper.xml 文件的路径-->  
        <!--resource 属性会自动从类的根路径下开始查找资源-->  
        <mapper resource="CarMapper.xml"/>  
    </mappers>  
</configuration>
```

注意：

- `mybatis` 核心配置文件的文件名不一定是 `mybatis-config.xml`，可以是其它名字
- `mybatis` 核心配置文件存放的位置也可以随意，这里选择放在 `resources` 根下，相当于放到了类的根路径下

****
## 4. 配置 XxxMapper.xml 文件

这里将 `CarMapper.xml` 与 `mybatis-config.xml` 放在同一个地方：

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="car">  
    <!--insert sql：保存一个汽车信息-->  
    <insert id="insertCar">  
        insert into t_car  
            (id,car_num,brand,guide_price,produce_time,car_type)        
        values            
	        (null,'102','丰田mirai',40.30,'2014-10-05','氢能源')  
    </insert>  
</mapper>
```

****
## 5. 整体流程

>每个基于 MyBatis 的应用都是以一个 `SqlSessionFactory` 的实例为核心的。`SqlSessionFactory` 的实例可以通过 `SqlSessionFactoryBuilder` 获得。而 `SqlSessionFactoryBuilder` 则可以从 XML 配置文件来构建出 `SqlSessionFactory` 实例

具体流程：

```
配置文件 → SqlSessionFactoryBuilder → SqlSessionFactory → SqlSession
```

1、`SqlSessionFactoryBuilder`：构建器（临时对象）

>用来读取 MyBatis 配置文件，构建 `SqlSessionFactory` ，用完即丢，不需要保存

通过 IO 流读取本地文件，调用`build()` 方法解析 XML 文件，然后返回一个 `SqlSessionFactory` 对象：

```java
InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
```

2、`SqlSessionFactory`：会话工厂（核心对象）

>创建 `SqlSession` 对象来执行数据库操作，管理连接

```java
SqlSession session = factory.openSession();  // 获取一个 SqlSession
```

3、`SqlSession`：数据库会话

>用于执行 SQL 操作（CRUD），每次的数据库操作都要创建一个新的 `SqlSession`

```java
SqlSession session = factory.openSession(); // 默认关闭自动提交
UserMapper mapper = session.getMapper(UserMapper.class); 
List<User> list = mapper.selectAll(); // 调用方法
session.close(); // 必须关闭
```

```
1. 加载配置文件（mybatis-config.xml）
         ↓
2. SqlSessionFactoryBuilder 解析 XML
         ↓
3. 构建 SqlSessionFactory（包含数据源、事务管理器、映射器等信息）
         ↓
4. 调用 factory.openSession() 获取 SqlSession
         ↓
5. 通过 SqlSession 获取 Mapper 接口代理
         ↓
6. 调用 mapper 中方法 → 执行 SQL → 返回结果
```

完整代码：

```java
public class MyBatisIntroductionTest {  
    public static void main(String[] args) {  
        SqlSession sqlSession = null;  
        try {  
            // 1.创建SqlSessionFactoryBuilder对象  
            SqlSessionFactoryBuilder sqlSessionFactoryBuilder = 
	        new SqlSessionFactoryBuilder();  
            // 2.创建SqlSessionFactory对象  
            SqlSessionFactory sqlSessionFactory =  
			sqlSessionFactoryBuilder
			.build(Resources.getResourceAsStream("mybatis-config.xml"));  
            // 3.创建SqlSession对象  
            SqlSession sqlSession = sqlSessionFactory.openSession();  
            // 4.执行SQL  
            int count = sqlSession.insert("insertCar");  
            System.out.println("更新了几条记录：" + count);  
            // 5.提交  
            sqlSession.commit();  
        } catch (Exception e) {  
            // 回滚  
            if (sqlSession != null) {  
                sqlSession.rollback();  
            }  
            e.printStackTrace();  
        } finally {  
            // 6.关闭  
            if (sqlSession != null) {  
                sqlSession.close();  
            }  
        }  
    }  
}
```

****
# 6. MyBatis 集成日志组件

>使用 SLF4J + Logback

1、引入相关依赖

```xml
<dependency>  
  <groupId>org.slf4j</groupId>  
  <artifactId>slf4j-api</artifactId>  
  <version>1.7.36</version>  
</dependency>  
<dependency>  
  <groupId>ch.qos.logback</groupId>  
  <artifactId>logback-classic</artifactId>  
  <version>1.2.11</version>  
</dependency>
```

2、配置 Logback 配置文件(resources/logback.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>  
  
<configuration debug="false">  
    <!-- 控制台输出 -->  
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->  
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>  
        </encoder>  
    </appender>  
    <!-- 按照每天生成日志文件 -->  
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!--日志文件输出的文件名-->  
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>  
            <!--日志文件保留天数-->  
            <MaxHistory>30</MaxHistory>  
        </rollingPolicy>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->  
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>  
        </encoder>  
        <!--日志文件最大的大小-->  
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">  
            <MaxFileSize>100MB</MaxFileSize>  
        </triggeringPolicy>  
    </appender>  
  
    <!--mybatis log configure-->  
    <logger name="com.apache.ibatis" level="TRACE"/>  
    <logger name="java.sql.Connection" level="DEBUG"/>  
    <logger name="java.sql.Statement" level="DEBUG"/>  
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>  
  
    <!-- 日志输出级别,logback日志级别包括五个：TRACE < DEBUG < INFO < WARN < ERROR -->  
    <root level="DEBUG">  
        <appender-ref ref="STDOUT"/>  
        <appender-ref ref="FILE"/>  
    </root>  
  
</configuration>
```

3、mybatis-config.xml 的日志配置为“显式优先”行为（可选）

只有在想要强制指定日志实现的时候才需要写，这不是必须的，默认自动识别即可用，很多项目都不配这一项：

```xml
<configuration>
  <settings>
    <setting name="logImpl" value="SLF4J"/>
  </settings>
</configuration>
```

此时控制台就会打印相应的日志信息，可以通过这些信息查看执行的 SQL 语句：

![](images/MyBatis%20入门程序/file-20250524201400.png)

****
# 7. MyBatis 工具类 SqlSessionUtil 的封装














