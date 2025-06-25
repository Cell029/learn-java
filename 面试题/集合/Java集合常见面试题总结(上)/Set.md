
# 1. Comparable 和 Comparator 的区别

[5.3 Comparable 接口和 Comparator 接口](../../../java笔记/Set%20集合.md#5.3%20Comparable%20接口和%20Comparator%20接口)

****
# 2. 无序性和不可重复性

- 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的，而哈希是确定性的，同样数据每次计算 hash 结果一样，因此结构内部规律固定，但表现出来无序。例如：`HashSet`、`HashMap`、`HashTable` 都体现了无序性。
- 不可重复性是指添加元素时先计算 `hashCode()` 确定位置，同位置若有元素，再调用 `equals()` 比较内容。需要同时重写 `equals()` 方法和 `hashCode()` 方法。例如：`HashSet`、`HashMap` 的 Key、以及 `LinkedHashSet` 都具有不可重复性。

****
# 3. 比较 HashSet、LinkedHashSet 和 TreeSet 的区别

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）；`LinkedHashSet` 的底层数据结构是双向链表和哈希表，维护插入顺序（即 FIFO）；`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和自定义 `Comparator` 排序。
- 底层数据结构不同又导致这三者的应用场景不同。`HashSet` 用于不需要保证元素插入和取出顺序的场景，只需快速去重（如缓存、集合运算）；`LinkedHashSet` 用于需保留插入顺序（如访问日志、LRU 缓存）；`TreeSet` 用于需自动排序或范围查询（如排行榜、字典序存储）。

****
