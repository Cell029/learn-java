
# 1. HashMap 和 HashTable 的区别

- **线程是否安全：** `HashMap` 是非线程安全的，`Hashtable` 是线程安全的，因为 `Hashtable` 内部的方法基本都经过 `synchronized` 修饰（如果要保证线程安全的话就使用 `ConcurrentHashMap` ）；
- **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰；
- **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；`Hashtable` 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
- **初始容量大小和每次扩充容量大小的不同：**
	1. 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16，之后每次扩充，容量变为原来的 2 倍。
	2. 创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小，以此减少哈希冲突（`HashMap` 中的`tableSizeFor()`方法保证）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
- **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间；`Hashtable` 则没有这样的机制。
- **哈希函数的实现**：`HashMap` 对哈希值进行了高位和低位的混合扰动处理以减少冲突，而 `Hashtable` 直接使用键的 `hashCode()` 值。

**`HashMap` 中带有初始容量的构造函数：**

```java
public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal initial capacity: " +
										   initialCapacity);
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " +
										   loadFactor);
	this.loadFactor = loadFactor;
	this.threshold = tableSizeFor(initialCapacity);
}
 public HashMap(int initialCapacity) {
	this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

此方法则保证了 `HashMap` 总是使用 2 的幂作为哈希表的大小：

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1; // 先减 1，避免 cap 本身刚好是 2 的幂，导致多扩容
    n |= n >>> 1; 
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

以上对 n 的操作是位运算，把 `n` 的二进制数中，高位 1 后面的所有位都变成 1，最终加 1 得到的就是大于等于原始 `cap` 的最小 2 的幂次方：

```text
假设 `cap = 5`：

cap - 1 = 4 → 0000 0100

n |= n >>> 1  → 0000 0110 // 先无符号右移 1 位，再按位或运算
n |= n >>> 2  → 0000 0111
n |= n >>> 4  → 0000 0111
n |= n >>> 8  → 0000 0111
n |= n >>> 16 → 0000 0111

n = 7
return n + 1 = 8
```

****
# 2. HashMap 和 HashSet 的区别

| `HashMap`                       | `HashSet`                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------ |
| 实现了 `Map` 接口                    | 实现 `Set` 接口，底层靠 `HashMap` 构成                                                         |
| 存储键值对                           | 仅存储对象，即单一元素                                                                          |
| 调用 `put()` 向 map 中添加元素          | 调用 `add()` 方法向 `Set` 中添加元素，实际上是对 `HashMap` 的 `put()` 方法的封装                           |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以 `equals()` 方法用来判断对象的相等性 |

****

# 3. HashMap 和 TreeMap 区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但 `TreeMap` 还实现了`NavigableMap`接口和`SortedMap` 接口，而实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力，例如：

- **定向搜索**: `ceilingEntry()`、 `floorEntry()`、 `higherEntry()` 和 `lowerEntry()` 等方法可以用于定位大于等于、小于等于、严格大于、严格小于给定键的最接近的键值对。
- **子集操作**: `subMap()`、`headMap()` 和 `tailMap()` 方法可以高效地创建原集合的子集视图，而无需复制整个集合。
- **逆序视图**:`descendingMap()` 方法返回一个逆序的 `NavigableMap` 视图，使得可以反向迭代整个 `TreeMap`。
- **边界操作**: `firstEntry()`、`lastEntry()`、`pollFirstEntry()` 和 `pollLastEntry()` 等方法可以方便地访问和移除元素。

这些方法都是基于红黑树数据结构的属性实现的，红黑树保持平衡状态，从而保证了搜索操作的时间复杂度为 O(log n)，这让 `TreeMap` 成为了处理有序集合搜索问题的强大工具。

```java
TreeMap<Integer, String> map = new TreeMap<>();  
map.put(1, "one");  
map.put(2, "two");  
map.put(3, "three");  
map.put(5, "five");  
  
System.out.println(map.ceilingEntry(3)); // 返回 ≥ 3 的最小键值对，3 
System.out.println(map.floorEntry(4));  // 返回 ≤ 3 的最大键值对，3 
System.out.println(map.higherEntry(3)); // 返回 > 3 的最小键值对，5
System.out.println(map.lowerEntry(3));  // 返回 < 3 的最大键值对，2

System.out.println(map.subMap(2, 5)); // 获取 [2, 5) 范围子集
System.out.println(map.headMap(3));  // 获取 < 3 范围子集
System.out.println(map.tailMap(3));  // 获取 ≥ from 范围子集

System.out.println(map); // {1=one, 2=two, 3=three, 5=five}
System.out.println(map.descendingMap()); // 返回逆序（大 -> 小）视图

System.out.println(map.firstEntry()); // 获取最小，1
System.out.println(map.lastEntry()); // 获取最大，5
System.out.println(map.pollFirstEntry()); // 获取最小并移除
System.out.println(map.pollLastEntry()); // 获取最大并移除
```

而实现`SortedMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力，默认是按 key 的升序排序，不过也可以指定排序的比较器。

****
# 4. HashSet 如何检查重复

>当把对象加入 `HashSet` 时，`HashSet`  会先计算对象的 `hashcode` 值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会调用 `equals()` 方法来检查 `hashcode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让加入操作成功。

关于 `HashSet` 的 `add()` 方法，它是对 `HashMap` 的 `put()` 方法的封装：

```java
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

调用 `HashMap.put(key, value)`，key 是元素 `e`，value 是 `PRESENT`（空对象，充当 `HashMap` 的 value），然后会进入 `put(key, value)` 进行重复值判断（`HashMap` 的），计算元素 e 的 hash 值，用于定位数组桶索引，然后根据哈希值找出数组中对应的位置，接着遍历该位置的链表或红黑树，判断在这里是否存在某个值的 key 与 传入的 e 相同，相同就不插入节点，而是更新节点的 value（而 e 的 value 是一个空对象，所以更不更无所谓，没什么影响）；如果不存在，则返回 e 的 value，也就是空对象 null，所以通过是否等于 null 来判断是否重复，true 为不重复，false 为重复。综上，可以看出 `HashSet` 的判重机制其实是依赖于 `HashMap` 的。

****
# 5. HashMap 的长度为什么是 2 的幂次方

[2.5 HashMap的容量一直是2的次幂](Set%20集合.md#2.5%20HashMap的容量一直是2的次幂)

****
# 6. HashMap 多线程操作导致死循环问题

JDK 1.7 及之前版本的 `HashMap` 在多线程环境下扩容操作可能存在死循环问题，当时的 `HashMap` 是基于数组 + 链表，当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，头插法可能会导致链表中的节点指向错误的位置，从而形成一个环形链表，进而使得查询元素的操作陷入死循环无法结束。

```text
老链表：table[2] -> Node1 -> Node2 -> Node3 -> null
```

此时两个线程同时扩容：线程 A 负责搬迁 Node1、Node2、Node3；线程B 也同时参与搬迁。而头插法的特性：

```java
Node<K, V> next = e.next;
int idx = ...; // 计算在新数组中的位置
e.next = newTable[idx];
newTable[idx] = e;
```

扩容开始时两个线程都创建了自己的新数组（newTable），此时线程 A 执行 `Node1` 的迁移：

```java
next = Node2  // 线程 A 先保存 Node1 的 next
newTable[idx] = Node1 -> null  // 线程A 把 Node1 迁移到新数组
```

恰巧此时线程 B 抢占执行，此时 A 还没有完成对老链表的操作，所以老链表的结构依然是 `table[2] -> Node1 -> Node2 -> Node3 -> null`，而 B 可能一口气就完成，此时的新链表结构为： `newTable[idx] = Node3 -> Node2 -> Node1 -> null`，老链表的结构此时就不存在了。

线程 B 执行完后，线程 A 继续执行 `Node2.next = newTable[idx]`，但此时 `newTable[idx]` 已经被线程 B 重写了，链表已经变化，所以结果就变成 `Node3 -> Node2 -> Node1 -> Node2 -> ...`，链表出现环。

为了解决这个问题，JDK 1.8 版本的 `HashMap` 采用了尾插法而不是头插法来避免链表倒置，并且所有节点 `next` 指针的修改发生在线程本地变量（`loTail`）中，直到链表完整后，一次性赋值完成迁移。但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在数据覆盖的问题，并发环境下，推荐使用 `ConcurrentHashMap` 。

****
# 7. HashMap 为什么线程不安全

JDK1.7 及之前版本，在多线程环境下，`HashMap` 扩容时会造成死循环和数据丢失的问题。JDK 1.8 后，在 `HashMap` 中，多个键值对可能会被分配到同一个桶（bucket），并以链表或红黑树的形式存储。多个线程对 `HashMap` 的 `put` 操作会导致线程不安全，具体来说会有数据覆盖的风险。

例如：

- 两个线程 1，2 同时进行 put 操作，并且发生了哈希冲突（hash 函数计算出的插入下标是相同的）。
- 不同的线程可能在不同的时间片获得 CPU 执行的机会，当前线程 1 执行完哈希冲突判断后，由于时间片耗尽挂起，而线程 2 可能先完成了插入操作。
- 随后，线程 1 获得时间片，由于之前已经进行过 hash 碰撞的判断，所有此时会直接进行插入，这就导致线程 2 插入的数据被线程 1 覆盖了。

****
# 8. HashMap 常见的遍历方式

1、使用迭代器（Iterator）EntrySet 的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e");   
Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();  
while (iterator.hasNext()) {  
	Map.Entry<Integer, String> entry = iterator.next();  
	System.out.println(entry.getKey() + ":" + entry.getValue());  
}
```

```text
1:a
2:b
3:c
4:d
5:e
```

2、使用迭代器（Iterator）KeySet 的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e");  
Iterator<Integer> iterator = map.keySet().iterator();  
while (iterator.hasNext()) {  
	Integer key = iterator.next();  
	System.out.println(key + ":" + map.get(key));   
}
```

3、使用 For Each EntrySet 的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e");  
for (Map.Entry<Integer, String> entry : map.entrySet()) {  
	System.out.println(entry.getKey() + ":" + entry.getValue());   
}
```

4、使用 For Each KeySet 的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e");   
for (Integer key : map.keySet()) {  
	System.out.println(key + ":" + map.get(key));  
}
```

5、使用 Lambda 表达式的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e"); 
map.forEach((key, value) -> {  
	System.out.println(key + ":" + value);  
});
```

6、使用 Streams API 单线程的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e"); 
map.entrySet().stream().forEach((entry) -> {  
	System.out.println(entry.getKey());  
	System.out.println(entry.getValue());  
});
```

7、使用 Streams API 多线程的方式进行遍历

```java
Map<Integer, String> map = new HashMap();  
map.put(1, "a");  
map.put(2, "b");  
map.put(3, "c");  
map.put(4, "d");  
map.put(5, "e"); 
map.entrySet().parallelStream().forEach((entry) -> {  
	System.out.println(entry.getKey());  
	System.out.println(entry.getValue());  
});
```

需要注意的是：存在阻塞时 parallelStream 性能最高, 非阻塞时因为涉及线程调度、分割、汇总等操作，parallelStream 反而没有普通运行的快

****
# 9. ConcurrentHashMap 和 Hashtable 的区别

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用分段的数组+链表实现，JDK1.8 采用的数据结构跟 1.8 的 `HashMap` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK 1.8 之前的 `HashMap` 的底层数据结构类似都是采用数组+链表的形式，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的；

- **实现线程安全的方式：**
    - 在 JDK1.7 的时候，`ConcurrentHashMap` 对整个桶数组进行了分割分段（使用 `Segment`，分段锁），每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
    - 到了 JDK1.8 的时候，`ConcurrentHashMap` 已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作（JDK1.6 以后 `synchronized` 锁做了很多优化） 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；
    - `Hashtable` 使用 `synchronized` 给方法加锁来保证线程安全，效率非常低。当一个线程调用该方法时，如果其他线程也想要调用该方法，那么就可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

****
# 10. ConcurrentHashMap 线程安全的具体实现方式

Java 7 之前使用 `Segment` 分段锁解决；Java 8 之后使用 `Node` + `CAS` + `synchronized` 解决

[ConcurrentHashMap 的线程安全问题](ConcurrentHashMap.md#4.%20ConcurrentHashMap%20的线程安全问题)

****
# 11. JDK 1.7 和 JDK 1.8 的 ConcurrentHashMap 实现有什么不同

[JDK 1.7 中的 ConcurrentHashMap](ConcurrentHashMap.md#2.%20JDK%201.7%20中的%20ConcurrentHashMap)

[JDK 1.8 中的 ConcurrentHashMap](ConcurrentHashMap.md#3.%20JDK%201.8%20中的%20ConcurrentHashMap)

****
# 12. ConcurrentHashMap 为什么 key 和 value 不能为 null

[ConcurrentHashMap 的 key 和 value 不能为 null](ConcurrentHashMap.md#3.3%20`ConcurrentHashMap`%20的%20key%20和%20value%20不能为%20null)

****
# 13. ConcurrentHashMap 能保证复合操作的原子性吗

