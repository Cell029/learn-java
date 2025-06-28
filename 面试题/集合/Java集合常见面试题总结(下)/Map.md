
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