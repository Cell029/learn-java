
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
# 5. 集合转数组

>使用集合转数组的方法，必须使用集合的 `toArray(T[] array)`，传入的是类型完全一致、长度为 0 的空数组。

`toArray(T[] array)` 方法的参数是一个泛型数组，如果 `toArray` 方法中没有传递任何参数的话返回的是 `Object`类 型数组。

```java
// 无参，返回 Object[] 类型
Object[] toArray()

// 传入一个泛型数组参数，返回 T[] 类型
<T> T[] toArray(T[] a)
```

```java
List<String> list = Arrays.asList("A", "B", "C");
String[] arr = (String[]) list.toArray(); // 编译不报错，但运行会抛异常
/*Exception in thread "main" java.lang.ClassCastException: class [Ljava.lang.Object; cannot be cast to class [Ljava.lang.String; ([Ljava.lang.Object; and [Ljava.lang.String; are in module java.base of loader 'bootstrap') at test2.main(test2.java:8)*/
```

这个方法返回的是 `Object[]` 类型的数组，而不是 `String[]`，虽然 Java 有类型擦除机制，但数组是协变且类型检查严格的对象，`Object[]` 不能直接强制转换为 `String[]`。

```java
String[] arr = list.toArray(new String[0]);
```

正确的做法是传入一个 `String[]` 类型，`toArray(T[] a)` 会尝试把 `List` 中的元素传入的数组 `a` 中。如果数组足够大，就直接使用这个数组；如果不够大，就新创建一个一样类型的新数组并返回。由于 JVM 优化，`new String[0]`作为 `Collection.toArray()` 方法的参数现在使用更好，`new String[0]` 就是起一个模板的作用，指定了返回数组的类型，0 是为了节省空间，因为它只是为了说明返回的类型。

****
# 6. 数组转集合

>使用工具类 `Arrays.asList()` 把数组转换成集合时，不能使用其修改集合相关的方法， 它的 `add/remove/clear` 方法会抛出 `UnsupportedOperationException` 异常。

`Arrays.asList()`在平时开发中还是比较常见的，可以使用它将一个数组转换为一个 `List` 集合。

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);
// 上面两个语句等价于下面一条语句
List<String> myList = Arrays.asList("Apple","Banana", "Orange");
```

而该方法接收一个变长参数（数组），并将其包装成一个固定长度的 `List`（因为底层结构为数组）

```java
public static <T> List<T> asList(T... a) {  
    return new ArrayList<>(a);  
}
```

使用注意事项：

1、`Arrays.asList()`是泛型方法，传递的数组必须是对象数组，而不是基本类型。

```java
int[] myArray = {1, 2, 3};
List myList = Arrays.asList(myArray);
System.out.println(myList.size());// 1
System.out.println(myList.get(0));// 数组地址值
System.out.println(myList.get(1));// 报错：ArrayIndexOutOfBoundsException:Index 1 out of bounds for length 1
int[] array = (int[]) myList.get(0);
System.out.println(array[0]);// 1
```

当传入一个原生数据类型数组时，`Arrays.asList()` 的真正得到的参数就不是数组中的元素，而是数组对象本身，此时 `List` 的唯一元素就是这个数组本身，所以获取第二个元素时报错超出范围。对列表的第一个元素（即数组本身）强转后，又变为普通的数组，所以可以获取每个元素。

```java
Integer[] myArray = {1, 2, 3};
```

但使用包装类后就不会出现上述问题，因为 Java 的泛型只支持引用类型（Object 的子类），不能直接用基本类型（如 `int`），所以`int[]` 作为一个基本类型数组，整体就被当成一个元素塞进 `List`(`T` 被推导为 `int[]`，不是 `int`，参数 `a` 实际是一个只有一个元素的数组：`new int[][] { arr }`)；而 `Integer[]` 是一个引用类型数组，会被自动拆分为多个元素，逐个填进 `List`。

2、使用集合的修改方法：`add()`、`remove()`、`clear()` 会抛出异常。

```java
List myList = Arrays.asList(1, 2, 3);
myList.add(4); // 运行时报错：UnsupportedOperationException
myList.remove(1); // 运行时报错：UnsupportedOperationException
myList.clear(); // 运行时报错：UnsupportedOperationException
```

`Arrays.asList()` 方法返回的并不是 `java.util.ArrayList` ，而是 `java.util.Arrays` 的一个内部类，这个内部类并没有实现集合的修改方法或者说并没有重写这些方法，它是一个定长、基于原始数组的只读视图集合，只能改元素，不支持改结构（增删）。

```java
public static <T> List<T> asList(T... a) {
    return new Arrays.ArrayList<>(a);
}

private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, Serializable {
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    public int size() {
        return a.length;
    }

    public E get(int index) {
        return a[index];
    }

    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }
}
```

3、正确的将数组转换为 `ArrayList`

1. 手动实现工具类

```java
static <T> List<T> arrayToList(final T[] array) {
  final List<T> l = new ArrayList<T>(array.length);
  for (final T s : array) {
    l.add(s);
  }
  return l;
}
Integer [] myArray = { 1, 2, 3 };
System.out.println(arrayToList(myArray).getClass()); // class java.util.ArrayList
```

通过将数组中的元素依次添加进 `ArrayList` 达到类似转换的效果。

2. 使用 ArrayList 的构造方法

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

`ArrayList` 的构造器中会先对传进来的集合进行判断，如果是相同的集合类型，则直接赋值；如果不是相同类型集合，则进行一次“复制”操作。如果原集合为空（`size == 0`），直接让内部数组引用共享的空数组对象 `EMPTY_ELEMENTDATA`，这是`ArrayList`类中定义的一个空数组常量，用于表示空的`ArrayList`。

```java
public ArrayList(Collection<? extends E> c) {  
    Object[] a = c.toArray();  
    if ((size = a.length) != 0) {  
        if (c.getClass() == ArrayList.class) {  
            elementData = a;  
        } else {  
            elementData = Arrays.copyOf(a, size, Object[].class);  
        }  
    } else {  
        elementData = EMPTY_ELEMENTDATA;  
    }  
}
```

3. 使用 Stream

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
// 基本类型也可以实现转换（依赖 boxed 的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

因为`Arrays.stream(int[])` 返回的是 `IntStream`，而不是 `Stream<Integer>`，所以必须通过 `boxed()` 把 `IntStream` 中的 `int` 转换为 `Integer` 对象，才能用 `Collectors.toList()` 收集为 `List<Integer>`。

4. 使用 Guava

对于不可变集合，你可以使用 `ImmutableList` 类及其 `of()` 与 `copyOf()` 工厂方法：（参数不能为空）

```java
List<String> il = ImmutableList.of("string", "elements");  // from varargs
List<String> il = ImmutableList.copyOf(aStringArray);      // from array
```

对于可变集合，你可以使用 `Lists` 类及其 `newArrayList()` 工厂方法：

```java
List<String> l1 = Lists.newArrayList(anotherListOrCollection); // from collection
List<String> l2 = Lists.newArrayList(aStringArray); // from array
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // from varargs
```

5. 使用 Apache Commons Collections

```java
List<String> list = new ArrayList<String>();
CollectionUtils.addAll(list, str);
```

****