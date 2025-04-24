
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

>集合的底层是重写了 `equals` 方法的，而 `remove(Object o)` 和 `contains(Object o)` 在使用时会调用 `equals` 方法的，所以只要传入的对象的内容相等（对象内部已重写 `equals` 方法），就会返回 `true`

****
## 3. Collection的通用迭代

>就是用统一的方式遍历 `Collection` 中的所有元素，而不依赖于集合的具体类型（如是否有索引等）

```java
Iterator it = collection.iterator();
```

>每次调用 `iterator()`，都会返回一个新的 `Iterator` 对象，它负责获取到 `collection` 集合的大小和当前遍历到的位置 `cursor` 以及上一个返回的元素下标 `lastRet`

![](images/Collection集合/file-20250424182009.png)

```java
iterator.hasNext()
```

>判断是否还有元素，它的底层代码会判断当前下标 `cursor` 是否等于集合的大小，不等为 ture 即为有下一个元素，相等证明此时的光标已经指向了集合中最后一个元素的下一个位置，代表已经遍历完集合

![](images/Collection集合/file-20250424182250.png)

```java
iterator.next()
```

>返回当前 `cursor` 指向的元素，返回的是一个数组元素 `elementData[lastRet = i]`

![](images/Collection集合/file-20250424182528.png)

```java
public void test2() {  
    Collection collection = new ArrayList();  
    collection.add("a");  
    collection.add("b");  
    collection.add("c");  
    collection.add("d");  
  
    Iterator iterator = collection.iterator();  
    while (iterator.hasNext()) {  
        System.out.println(iterator.next()); // a b c d  
    }  
}
```

****
## 4. SequencedCollection

