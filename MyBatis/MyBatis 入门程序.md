
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
            sqlSession = sqlSessionFactory.openSession();  
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







