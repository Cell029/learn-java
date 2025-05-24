
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

## 4.1 查询一条语句

