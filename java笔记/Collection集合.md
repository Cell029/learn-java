
![](images/Collection集合/file-20250424165521.png)

## 1. Collection接口

>`Collection` 是一个接口，位于 `java.util` 包中，它是单列集合的根接口，表示一组元素的集合，是接口，不可以直接实例化，必须使用它的子接口或实现类，如 `List`、`Set`

**单列集合**

>只有值没有键的集合

****
## 2. 常用方法

| 方法名                                 | 功能              |
| ----------------------------------- | --------------- |
| `add(E e)`                          | 添加一个元素          |
| `addAll(Collection<? extends E> c)` | 添加一个集合的所有元素     |
| `remove(Object o)`                  | 删除指定元素          |
| `removeAll(Collection<?> c)`        | 删除所有与参数集合中相同的元素 |
| `retainAll(Collection<?> c)`        | 只保留与参数集合中相同的元素  |
| `clear()`                           | 清空集合            |
| `contains(Object o)`                | 判断是否包含某个元素      |
| `containsAll(Collection<?> c)`      | 判断是否包含集合中所有元素   |
| `isEmpty()`                         | 判断集合是否为空        |
| `size()`                            | 返回集合大小          |
| `iterator()`                        | 返回迭代器，用于遍历      |
| `toArray()`                         | 转为 Object 数组    |
| `toArray(T[] a)`                    | 转为指定类型的数组       |

>

****
## 3. Collection的通用迭代

