
# 1. Quequ 与 Deque 的区别

`Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循先进先出（FIFO）规则，它扩展了 `Collection` 的接口，根据因为容量问题而导致操作失败后处理方式的不同可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

1. **插入操作**
    
    - `add(E e)`：插入元素，若队列已满（对于有界队列），抛出异常 `IllegalStateException`
    - `offer(E e)`：尝试插入，若队列已满，返回 `false`，不会抛出异常。

2. **获取队头元素（不移除）**
    
    - `element()`：获取队头元素，若队列为空，抛出 `NoSuchElementException`。
    - `peek()`：获取队头元素，若队列为空，返回 `null`。
    
3. **获取并移除队头元素**
    
    - `remove()`：移除并返回队头元素，若队列为空，抛出 `NoSuchElementException`。
    - `poll()`：移除并返回队头元素，若队列为空，返回 `null`。

`Deque` 是双端队列，在队列的两端均可以插入或删除元素，它扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

1.  **插入操作**

	- `addFirst(E e)`：插入队首，若队列已满，抛出 `IllegalStateException`
	- `offerFirst(E e)`：尝试插入队首，若队列已满，返回 `false`
	- `addLast(E e)`：插入队尾，若队列已满，抛出 `IllegalStateException`
	- `offerLast(E e)`：尝试插入队尾，若队列已满，返回 `false`

2. **获取队首/队尾元素（不移除）**

	- `getFirst()`：获取队首元素，若队列为空，抛出 `NoSuchElementException`
	- `peekFirst()`：获取队首元素，若队列为空，返回 `null`
	- `getLast()`：获取队尾元素，若队列为空，抛出 `NoSuchElementException`
	- `peekLast()`：获取队尾元素，若队列为空，返回 `null`

3. **获取并移除队首/队尾元素**

	- `removeFirst()`：移除并返回队首元素，若队列为空，抛出 `NoSuchElementException`
	- `pollFirst()`：移除并返回队首元素，若队列为空，返回 `null`
	- `removeLast()`：移除并返回队尾元素，若队列为空，抛出 `NoSuchElementException`
	- `pollLast()`：移除并返回队尾元素，若队列为空，返回 `null`

****
# 2. ArrayDeque 和 LinkedList 的区别

- `ArrayDeque` 是基于可变长的环形数组，通过双指针（head、tail）管理队首队尾的位置，而 `LinkedList` 则通过链表来实现。
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持。
- `ArrayDeque` 是在 JDK 1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在。
- `ArrayDeque` 插入时可能存在扩容过程（只支持头插和尾插），不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

****