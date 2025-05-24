
# 1. INSERT

## 1.1 使用 Map 集合进行传参

```java
// 准备数据  
Map<String, Object> map = new HashMap<>();  
map.put("carNum", "103");  
map.put("brand", "奔驰E300L");  
map.put("guidePrice", 50.3);  
map.put("produceTime", "2020-10-01");  
map.put("carType", "燃油车");  
// 获取SqlSession对象  
SqlSession sqlSession = SqlSessionUtil.openSession();  
// 执行SQL语句
// 第一个参数：sqlId；
// 第二个参数：封装数据的对象，将对象中的数据映射到 sql 语句的占位符中
int count = sqlSession.insert("insertCar", map);  
sqlSession.commit();  
sqlSession.close();  
System.out.println("插入了几条记录：" + count);
```

SQL 中使用 `#{key}` 获取对应的键值：

```xml
<mapper namespace="car">  
    <insert id="insertCar">  
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})  
    </insert>  
</mapper>
```

>如果占位符中写的与 Map 中的 key 不同的话并不会导致程序出错，但是会导致插入的数据为 Null

****
## 1.2 使用 POJO 类进行传参

>使用这种方法需要创建对应的 POJO 实体类，每个字段的名称要与数据库中的对应，以此提高可读性

```java
Car car = new Car();  
car.setCarNum("104");  
car.setBrand("奔驰C200");  
car.setGuidePrice(33.23);  
car.setProduceTime("2020-10-11");  
car.setCarType("燃油车");  
// 获取SqlSession对象  
SqlSession sqlSession = SqlSessionUtil.openSession();  
// 执行SQL，传数据  
int count = sqlSession.insert("insertCarByPOJO", car);  
sqlSession.commit();  
sqlSession.close();  
System.out.println("插入了几条记录" + count);
```

>需要注意的是：这里的占位符中的 key 是必须和 POJO 实体类的 getter() 方法的名字一样（去掉 get 然后首字母小写），否则就会因为找不到 getter 方法而无法传递参数（如果修改了 getter 方法的名字，那么传参时也要修改成对应的）

```xml
<mapper namespace="car">  
    <insert id="insertCarByPOJO">  
        <!--#{} 里写的是POJO的属性名-->  
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})  
    </insert>  
</mapper>
```

![](images/MyBatis%20的%20CRUD/file-20250524212802.png)

****
# 2. DELETE

```java 
SqlSession sqlSession = SqlSessionUtil.openSession();  
int count = sqlSession.delete("deleteByCarNum", "102");  
sqlSession.commit();  
sqlSession.close();  
System.out.println("删除了几条记录：" + count);
```

>当占位符只有一个的时候，${} 里面的内容可以随便写，不过还是建议和对象的字段保持一致

```xml
<delete id="deleteByCarNum">  
    delete from t_car where car_num = #{xx}  
</delete>
```

****
# 3. UPDATE

```java 
Car car = new Car();  
car.setId(12L);  
car.setCarNum("102");  
car.setBrand("比亚迪汉");  
car.setGuidePrice(30.23);  
car.setProduceTime("2018-09-10");  
car.setCarType("电车");  
SqlSession sqlSession = SqlSessionUtil.openSession();  
int count = sqlSession.update("updateCarByPOJO", car);  
sqlSession.commit();  
sqlSession.close();  
System.out.println("更新了几条记录：" + count);
```

```xml
<update id="updateCarByPOJO">  
    update t_car set car_num = #{carNum}, brand = #{brand},guide_price = #{guidePrice},produce_time = #{produceTime},car_type = #{carType} where id = #{id}</update>
```

****
# 4. SELECT（RETRIVE）

>在 MyBatis 中，`<select>` 语句会返回一个结果集（resultset），为了让 MyBatis 能够正确地把数据库中的查询结果映射为 Java 对象，必须告诉它结果类型或映射关系，这就涉及到 `resultType` 或 `resultMap` 的配置

## 4.1 `resultType`

`resultType` 是 MyBatis `<select>` 语句中用来指定查询结果返回值类型的属性：

>它通常用于将查询结果集自动封装为某个 Java 类型（比如基本类型、pojo、Map 等），本质上 MyBatis 会根据数据库返回的列名，去反射调用 Java 对象中对应的 setter 方法，然后将数据库中查询到的记录封装进对象

resultType 的字段映射规则：

- 数据库字段名和 Java 属性名一致（或符合驼峰转换规则）时自动映射；
- 数据库字段名为 `car_num`，Java 属性名为 `carNum`，也可以映射（前提是开启了驼峰命名自动映射）：

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

>当字段名和属性名差异太大；涉及嵌套对象、多表联查；需要更精确地控制字段到属性的映射时，就不适合用 `resultType`，需要使用 `<resultMap>`

****
## 4.2 查询一条数据

```java
SqlSession sqlSession = SqlSessionUtil.openSession();  
Object car = sqlSession.selectOne("selectCarById", 1);  
System.out.println(car);
```

```xml
<select id="selectCarById" resultType="com.cell.pojo.Car">  
    select * from t_car where id = #{id}  
</select>
```

此时的打印结果是这样的，只有 id 和 brand 字段有值，其余为空，这就是因为 Java 底层通过数据库的列名进行映射时没有找到该对象的 setter 方法导致的，所以要么查询时起别名，要么添加驼峰命名自动映射组件

```java
Car{id=1, carNum='null', brand='宝马520Li', guidePrice=null, produceTime='null', carType='null'}
```

****
## 4.2 查询多条数据

>使用 `selectList` 方法将查询到的多个对象封装成集合，然后遍历打印

```java
SqlSession sqlSession = SqlSessionUtil.openSession();  
List<Object> cars = sqlSession.selectList("selectCarAll");  
cars.forEach(car -> System.out.println(car));
```

```xml
<!--虽然结果是List集合，但是resultType属性需要指定的是List集合中元素的类型。-->  
<select id="selectCarAll" resultType="com.cell.pojo.Car">   
    select * from t_car  
</select>
```

****





