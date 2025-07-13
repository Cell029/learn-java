
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



