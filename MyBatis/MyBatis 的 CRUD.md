
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

****
## 1.2 使用 POJO 类进行传参

