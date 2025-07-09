
# 1. 相关锁

- [synchronized](多线程.md#12.%20synchronized)

Java 中的关键字，内部实现为监视器锁，主要是通过对象监视器在对象头中的字段来表明的。synchronized 从旧版本到现在已经做了很多优化了，在运行时会有三种存在方式：偏向锁，轻量级锁，重量级锁。

- 偏向锁

是指一段同步代码一直被一个线程访问，那么这个线程会自动获取锁，降低获取锁的代价。

- 轻量级锁

是指当锁是偏向锁时，被另一个线程所访问，偏向锁会升级为轻量级锁，这个线程会通过自旋的方式尝试获取锁，不会阻塞，提高性能。

- 重量级锁

是指当锁是轻量级锁时，当自旋的线程自旋了一定的次数后，还没有获取到锁，就会进入阻塞状态，该锁升级为重量级锁，重量级锁会使其他线程阻塞，性能降低。

- CAS

`Compare And Swap`，一种乐观锁，认为对于同一个数据的并发操作不一定会发生修改，在更新数据的时候，尝试去更新数据，如果失败就不断尝试。

- volatile（非锁）

java中的关键字，当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

- 自旋锁

自旋锁，是指尝试获取锁的线程不会阻塞，而是循环的方式不断尝试，这样的好处是减少线程的上下文切换带来的开锁，提高性能，缺点是循环会消耗 CPU。

- 分段锁

分段锁，是一种锁的设计思路，它细化了锁的粒度，主要运用在 ConcurrentHashMap 中，实现高效的并发操作，当操作不需要更新整个数组时，就只锁数组中的一项就可以了。

- ReentrantLock

可重入锁，是指一个线程获取锁之后再尝试获取锁时会自动获取锁，可重入锁的优点是避免死锁。其实，synchronized 也是可重入锁。

****
# 2. JDK 1.7 中的 ConcurrentHashMap

## 2.1 概念

Segment 本身就相当于一个 HashMap 对象，Segment 包含一个 HashEntry 数组，数组中的每一个 HashEntry 既是一个键值对，也是一个链表的头节点。

ConcurrentHashMap 的核心成员：

```java
// Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
// 逻辑上相当于分段的 HashMap，每个 Segment 管一个子表
// 可以类比成 HashMap 中的一个“大桶”被切分成了多个“小桶”，每个 Segment 就是一个小 HashMap，锁粒度被降到了 Segment 级别。
final Segment<K,V>[] segments;
transient Set<K> keySet;
transient Set<Map.Entry<K,V>> entrySet;
```

Segment 是 ConcurrentHashMap 的一个内部类，主要的组成如下：

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
	private static final long serialVersionUID = 2249069246763182397L;
	transient volatile HashEntry<K,V>[] table;
	transient int count; // 当前元素数量
	transient int modCount; // 修改次数（用于 fail-fast）
	transient int threshold; // 扩容门槛
	final float loadFactor; // 负载因子
}
```

```java
// 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
// 但 value ，以及链表都是 volatile 修饰的
static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;
}
```

原理上来说：ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock，每个  Segment 就是一个可重入锁，每个 Segment 内部的结构和 `HashMap` 类似，但对并发做了优化，所以不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel （Segment 数组数量，默认为 16）的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

单一的 Segment 结构如下：

![](images/ConcurrentHashMap/file-20250709180653.png)

像这样的 Segment 对象，在 ConcurrentHashMap 集合中有 2<sup>N</sup> 次方个，共同保存在一个名为segments 的数组当中， 因此整个 ConcurrentHashMap 的结构如下：

![](images/ConcurrentHashMap/file-20250709180735.png)

所以 ConcurrentHashMap 可以看作是一个二级哈希表，在一个总的哈希表下面，有多个子哈希表，解决哈希冲突的办法与 HashMap 也一致。

并发读写例如：

1、不同 Segment 的并发写入（可以并发执行）

![](images/ConcurrentHashMap/file-20250709183122.png)

2、同一Segment的一写一读（可以并发执行）

![](images/ConcurrentHashMap/file-20250709183139.png)

3、同一 Segment 的并发写入

![](images/ConcurrentHashMap/file-20250709183241.png)

****
## 2.2 get 与 put 方法

**put 方法**：

```java
public V put(K key, V value) {
	Segment<K,V> s;
	if (value == null)
		throw new NullPointerException();
	int hash = hash(key);
	int j = (hash >>> segmentShift) & segmentMask; // 定位到具体的 Segment[j] 桶
	if ((s = (Segment<K,V>)UNSAFE.getObject          
		 (segments, (j << SSHIFT) + SBASE)) == null) 
		s = ensureSegment(j);
	return s.put(key, hash, value, false); // 此时才是调用真正的 put 方法，进入加锁写入流程
}
```

首先是在外层的 put 方法中通过 key 定位到 Segment，之后在对应的 Segment 中进行具体的 put。首先第一步的时候会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 `scanAndLockForPut()` 自旋获取锁。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
	// 优先用 tryLock() 尝试获取锁，避免阻塞
	HashEntry<K,V> node = tryLock() ? null :
		scanAndLockForPut(key, hash, value);
	V oldValue;
	try {
		HashEntry<K,V>[] tab = table;
		// 该操作与 HashMap 中的相似，通过该操作找到桶头节点 first，准备遍历链表
		int index = (tab.length - 1) & hash;
		HashEntry<K,V> first = entryAt(tab, index);
		for (HashEntry<K,V> e = first;;) {
			// 判断 key 是否存在
			if (e != null) {
				K k;
				// 如果 key 存在
				if ((k = e.key) == key ||
					(e.hash == hash && key.equals(k))) {
					oldValue = e.value;
					// 此时判断是否 onlyIfAbsent 是否为 false，是则选择覆盖旧值，否则就跳过
					if (!onlyIfAbsent) {
						e.value = value;
						++modCount;
					}
					break;
				}
				e = e.next;
			}
			else { // key 不存在，准备插入新节点
				if (node != null)
					node.setNext(first);
				else
					// 使用头插法创建节点
					node = new HashEntry<K,V>(hash, key, value, first);
				int c = count + 1;
				if (c > threshold && tab.length < MAXIMUM_CAPACITY)
					// 判断是否需要进行 rehash 进行扩容
					rehash(node);
				else
					setEntryAt(tab, index, node);
				++modCount;
				count = c;
				oldValue = null;
				break;
			}
		}
	} finally {
		unlock();
	}
	return oldValue;
}
```

整体流程：

1. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
2. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
3. 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
4. 最后会解除在 1 中所获取当前 Segment 的锁。

**get 方法**：



```java
public V get(Object key) {
	Segment<K,V> s; 
	HashEntry<K,V>[] tab;
	int h = hash(key);
	long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
	if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
		(tab = s.table) != null) {
		for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
				 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
			 e != null; e = e.next) {
			K k;
			if ((k = e.key) == key || (e.hash == h && key.equals(k)))
				return e.value;
		}
	}
	return null;
}
```

