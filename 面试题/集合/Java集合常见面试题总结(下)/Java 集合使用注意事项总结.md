
# 1. 集合判空

>判断所有集合内部的元素是否为空，使用 `isEmpty()` 方法，而不是 `size() == 0` 的方式。

这是因为 `isEmpty()` 方法的可读性更好，并且时间复杂度为 `O(1)`。绝大部分我们使用的集合的 `size()` 方法的时间复杂度也是 `O(1)`，例如`ArrayList`、`HashSet` 的 `size()` 方法是 `O(1)`。不过，也有很多复杂度不是 `O(1)` 的，比如 `java.util.concurrent` 包下的 `ConcurrentLinkedQueue`。`ConcurrentLinkedQueue` 的 `isEmpty()` 方法通过 `first()` 方法进行判断，其中 `first()` 方法返回的是队列中第一个值不为 `null` 的节点（节点值为 `null` 的原因是在迭代器中使用的逻辑删除），所以该方法的执行的时间复杂度可以近似为 `O(1)`。而 `size()` 方法需要遍历整个链表，时间复杂度为`O(n)`。

```java
public int size() {
    int count = 0;
    for (Node<E> p = first(); p != null; p = succ(p))
        if (p.item != null)
            if (++count == Integer.MAX_VALUE)
                break;
    return count;
}
```

此外，在 Java 7 的 `ConcurrentHashMap`  中 `size()` 方法和 `isEmpty()` 方法的时间复杂度也不太一样。`ConcurrentHashMap` 1.7 将元素数量存储在每个`Segment` 中，`size()` 方法需要统计每个 `Segment` 的数量，而 `isEmpty()` 只需要找到第一个不为空的 `Segment` 即可。但是在 Java 8 的 `ConcurrentHashMap` 中的 `size()` 方法和 `isEmpty()` 都需要调用 `sumCount()` 方法（判断总数是否为 0），其时间复杂度与 `Node` 数组的大小有关。

```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null)
        for (int i = 0; i < as.length; ++i)
            if ((a = as[i]) != null)
                sum += a.value;
    return sum;
}
```

这是因为在并发的环境下，`ConcurrentHashMap` 将每个 `Node` 中节点的数量存储在 `CounterCell[]` 数组中，如果多线程频繁修改同一个共享计数器容易导致性能瓶颈，所以使用这种特性允许线程分别维护自己的计数单元。

****
# 2. 集合转 Map

>在使用 `java.util.stream.Collectors` 类的 `toMap()` 方法转为 `Map` 集合时，一定要注意当 `value` 为 `null` 时会抛 `NullPointerException` 异常。

```java
public static <T, K, U>  
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,  
                                Function<? super T, ? extends U> valueMapper) {  
    return new CollectorImpl<>(HashMap::new,  
                               uniqKeysMapAccumulator(keyMapper, valueMapper),  
                               uniqKeysMapMerger(),  
                               CH_ID);  
}
```

```java
BiConsumer<Map<K, V>, T> uniqKeysMapAccumulator(Function<? super T, ? extends K> keyMapper,  
                                                Function<? super T, ? extends V> valueMapper) {  
    return (map, element) -> {  
        K k = keyMapper.apply(element);  
        V v = Objects.requireNonNull(valueMapper.apply(element));  
        V u = map.putIfAbsent(k, v);  
        if (u != null) throw duplicateKeyException(k, u, v);  
    };  
}
```

```java
@ForceInline  
public static <T> T requireNonNull(T obj) {  
    if (obj == null)  
        throw new NullPointerException();  
    return obj;  
}
```

可以看到 `toMap()` 方法内部会对 `value` 进行判断，如果为空则直接抛出 `NullPointerException`。所以建议按照如下规则使用：

1、key 不重复

```java
// 1. 基本用法（key/value 映射）
toMap(Function<T, K> keyMapper, Function<T, V> valueMapper)
```

```java
List<String> list = List.of("a", "bb", "ccc");
Map<Integer, String> map = list.stream()
        .collect(Collectors.toMap(
                String::length, // keyMapper：用字符串长度作为 key
                s -> s // valueMapper：原字符串作为 value
        ));
System.out.println(map); // {1=a, 2=bb, 3=ccc}
```

需要注意的是：如果 key 有重复（如两个字符串长度一样），会抛出 `IllegalStateException: Duplicate key`，所以建议填写第三个参数 `mergeFunction`。

2、指定合并函数，解决 key 冲突

```java
// 2. key 冲突时的合并函数
toMap(Function<T, K> keyMapper, Function<T, V> valueMapper, BinaryOperator<V> mergeFunction)
```

```java
List<String> list = List.of("a", "aa", "b");
Map<Integer, String> map = list.stream()
        .collect(Collectors.toMap(
                String::length, // key：字符串长度
                s -> s, // value：原字符串
                (v1, v2) -> v1 + v2 // mergeFunction：合并相同 key 的 value
        ));
System.out.println(map); // {1=ab, 2=aa}
```

3、自定义返回 Map 类型（如 TreeMap）

```java
// 3. key 冲突 + 自定义 map 类型（如 TreeMap）
toMap(Function<T, K> keyMapper, Function<T, V> valueMapper, BinaryOperator<V> mergeFunction, Supplier<M> mapFactory)
```

```java
List<String> list = List.of("a", "b", "cc");
Map<Integer, String> treeMap = list.stream()
        .collect(Collectors.toMap(
                String::length,
                s -> s,
                (v1, v2) -> v2, // 当 key 相同时取最新的
                TreeMap::new // 指定用 TreeMap 存储
        ));
System.out.println(treeMap); // {1=b, 2=cc}
```

****
# 3. 集合遍历

>不要在 foreach 循环里进行元素的 `remove/add` 操作。remove 元素请使用 `Iterator` 方式，如果并发操作，需要对 `Iterator` 对象加锁。

[并发修改问题](Collection集合.md#6.%20并发修改问题)

****
# 4. 集合去重

>可以利用 `Set` 元素唯一的特性，可以快速对一个集合进行去重操作，避免使用 `List` 的 `contains()` 进行遍历去重或者判断包含操作。

```java
// Set 去重代码示例
public static <T> Set<T> removeDuplicateBySet(List<T> data) {
    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);
}

// List 去重代码示例
public static <T> List<T> removeDuplicateByList(List<T> data) {
    if (CollectionUtils.isEmpty(data)) {
        return new ArrayList<>();
    }
    List<T> result = new ArrayList<>(data.size());
    for (T current : data) {
        if (!result.contains(current)) {
            result.add(current);
        }
    }
    return result;
}
```

两者的核心差别在于 `contains()` 方法的实现。`HashSet` 的 `contains()` 方法底部依赖的 `HashMap` 的 `containsKey()` 方法，时间复杂度接近于 O(1)（没有出现哈希冲突的时候为 O(1)）。

```java
private transient HashMap<E,Object> map;
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

```java
public boolean containsKey(Object key) {  
    return getNode(key) != null;  
}

final Node<K,V> getNode(Object key) {
    int hash = hash(key); // 计算 hash 值
    int index = (n - 1) & hash; // 计算数组索引
    Node<K,V> first = table[index]; // 获取桶的第一个节点
    if (first != null) {
        // 如果第一个节点就命中，直接返回
        if (first.hash == hash && (first.key.equals(key))) return first;
        // 如果是红黑树结构
        if (first instanceof TreeNode) {
            return ((TreeNode<K,V>)first).getTreeNode(hash, key);
        }
        // 否则链表遍历
        Node<K,V> e = first.next;
        while (e != null) {
            if (e.hash == hash && (e.key.equals(key))) return e;
            e = e.next;
        }
    }
    return null;
}
```

可以看到底层是通过哈希函数定位桶，如果桶里只有一个元素，那么时间复杂度就是 `O(1)`；但当哈希冲突严重时，所有数据都落在同一个桶，时间复杂度就变为 `O(n)`，但基本情况是根据使用的是链表还是红黑树而定的。Java 8 之后，当桶中元素数超过阈值（默认为 8），会转为红黑树结构，从而避免链表导致的查找效率低下。所以时间复杂度为 `O(1)` ~ `O(logn)`。

而`ArrayList` 的 `contains()` 方法是通过遍历所有元素的方法来做的，时间复杂度接近是 O(n)。

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

所以从效率上来看，推荐使用 Set 集合去重。

****