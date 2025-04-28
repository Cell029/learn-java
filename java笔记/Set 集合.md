
>`Set` 不允许重复元素，即每个元素在集合中都是唯一的， `Set` 不保证元素的插入顺序，也不保证元素的排序，`Set` 是一个接口，它的常见实现类包括 `HashSet`（无序不重复）、`LinkedHashSet` （插入有序不充分）和 `TreeSet` （大小有序不重复）

****
# 1. Map 接口

![](images/Set%20集合/file-20250428133532.png)

> `Map` 接口和 `Set` 接口明面上没什么关系，但是 `HashSet`、`LinkedHashSet` 和 `TreeSet` 的底层是靠 `HashMap` 实现的，所以实际上是存在一定关系的

****
## 1.1 定义

>`Map` 是参数化类型，有两个类型变量。类型变量 K 表示映射中键的类型，类型变量 V 表示键对应的值的类型，虽然概念上 `Map` 像是一种集合，但在 Java 设计上，它是独立的，不是 `Collection`，因为 `Map` 中的键都是唯一的，`Set`也是一个元素唯一的集合，所以映射的键可以看成 `Set` 对象，映射的值可以看成 `Collection` 对象，而映射的键值对可以看成由 `Map.Entry` 对象组成的 `Set` 对象

****
### 1.1.1 键 Key

![](images/Set%20集合/file-20250428141802.png)

>`HashMap` 内部有一个 `keySet` 字段，第一次调用时才创建一个新的 `KeySet` 实例，`KeySet` 接收的就是 `Map` 集合中的键，这并不是普通的复制关系，而是返回一个只允许移除元素的视图，所以是不饿能直接通过 `KeySet` 对象修改或新增键的，因为 `Map` 的键值是绑定在一起的，所以只能通过 `keySet()` 删掉旧键，重新加一个新键

```java
Set<Integer> integers = map.keySet(); // 是用 Set 接收 Key 
System.out.println(integers);
```

****
### 1.1.2 值 Value

>每个键只对应一个值，但是不同键可以指向相同的值，所以 `Value` 才可以看作是集合

![](images/Set%20集合/file-20250428144016.png)

>这个方法的整体操作和上面的 `KeySet` 类似，也是作为一个视图，可以直接通过 `Values` 对象删除值（整个键值对删除），但是不能修改或增加，这需要通过 `Map` 对象来操作

****
## 2. 迭代方式

### 2.1 迭代器 + while

```java
Iterator<Integer> iterator = keys.iterator();  
while (iterator.hasNext()) {  
    Integer key = iterator.next();  
    String value = map.get(key);  
    System.out.println(key + "=" + value);  
}
```

>在 `HashMap` 中也存在迭代器，通过迭代器来获取 `Set` 集合中的所有键，然后再根据键获取对应的值

****
### 2.2 迭代器 + 增强 for

```java
for (Integer key : keys) {  
    String value = map.get(key);  
    System.out.println(key + "=" + value);  
}
```

>这种遍历方式虽然没有直接调用迭代器，但是它的底层会自动调用，把迭代器获取的 `next` 赋值给 `key` ，但是如果集合内部是不带构造器的就不能这样使用了

****
### 2.3 `Map.Entry` 接口

>`Map.Entry` 是用来表示 `Map` 中“一个键值对”的对象

```java
Set<Map.Entry<Integer, String>> entries = map.entrySet();  
Iterator<Map.Entry<Integer, String>> iterator = entries.iterator();  
while (iterator.hasNext()) {  
    Map.Entry<Integer, String> entry = iterator.next();  
    System.out.println(entry.getKey() + "=" + entry.getValue());  
}
```

>把 `Map` 中的键值对封装成一个对象，让他们实现简单的统一，一个对象就代表着一条数据，当需要修改数据时直接通过对象修改，降低了键值对与 `Map` 直接的耦合度，让管理变得更加方便，因为每次 `new` 对象都是不一样的，所有也很符合 `Set` 集合的特性，就可以直接把 `Entry` 对象放进去，并且封装的属性获取更方便，不管拿到的是 `key` 还是 `value`，它们一定是互相对应的，所以就可以实现单独的修改，那么 `Entry` 类里面就可以封装很多的方法提供给外部使用

****

#### 2.3.1 