
>`Set` 不允许重复元素，即每个元素在集合中都是唯一的， `Set` 不保证元素的插入顺序，也不保证元素的排序，`Set` 是一个接口，它的常见实现类包括 `HashSet`（无序不重复）、`LinkedHashSet` （插入有序不充分）和 `TreeSet` （大小有序不重复）

****
# 1. Map 接口

![](images/Set%20集合/file-20250428133532.png)

> `Map` 接口和 `Set` 接口明面上没什么关系，但是 `HashSet`、`LinkedHashSet` 和 `TreeSet` 的底层是靠 `HashMap` 实现的，所以实际上是存在一定关系的