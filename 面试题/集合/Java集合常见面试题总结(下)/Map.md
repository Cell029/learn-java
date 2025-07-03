
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



