
>`Set` 不允许重复元素，即每个元素在集合中都是唯一的， `Set` 不保证元素的插入顺序，也不保证元素的排序，`Set` 是一个接口，它的常见实现类包括 `HashSet`（无序不重复）、`LinkedHashSet` （插入有序不充分）和 `TreeSet` （大小有序不重复）

****
# 1. Map 接口

![](images/Set%20集合/file-20250428133532.png)

> `Map` 接口和 `Set` 接口明面上没什么关系，但是 `HashSet`、`LinkedHashSet` 和 `TreeSet` 的底层是靠 `HashMap` 实现的，所以实际上是存在一定关系的

****
## 1.1 定义

>`Map` 是参数化类型，有两个类型变量。类型变量 K 表示映射中键的类型，类型变量 V 表示键对应的值的类型，虽然概念上 `Map` 像是一种集合，但在 Java 设计上，它是独立的，不是 `Collection`，因为 `Map` 中的键都是唯一的，`Set`也是一个元素唯一的集合，所以映射的键可以看成 `Set` 对象，映射的值可以看成 `Collection` 对象，而映射的键值对可以看成由 `Map.Entry` 对象组成的 `Set` 对象

****
### 1.1.1 键 Key

![](images/Set%20集合/file-20250428141802.png)

>`HashMap` 内部有一个 `keySet` 字段，第一次调用时才创建一个新的 `KeySet` 实例，`KeySet` 接收的就是 `Map` 集合中的键，这并不是普通的复制关系，而是返回一个只允许移除元素的视图，所以是不饿能直接通过 `KeySet` 对象修改或新增键的，因为 `Map` 的键值是绑定在一起的，所以只能通过 `keySet()` 删掉旧键，重新加一个新键

```java
Set<Integer> integers = map.keySet(); // 是用 Set 接收 Key 
System.out.println(integers);
```

![](images/Set%20集合/file-20250428162706.png)

> `KeySet` 是一个懒加载的集合，它是 `Map` 的视图，并不直接存储所有的 `key`，可以通过内部的迭代器动态地从 `Map` 中获取所有的 `key`，需要注意的是，并不是一创建了 `KeySet` 对象就调用了迭代器获取所有的 `key` ，而是在程序员手动遍历 `key` 时才会自动调用它内部的迭代器动态获取，因为使用是的迭代器来获取元素，所以实际是通过迭代器来实现修改的操作的

>可以通过 `System.out.println()` 来验证是否是这样的运行逻辑

![](images/Set%20集合/file-20250428164435.png)

![](images/Set%20集合/file-20250428164451.png)

![](images/Set%20集合/file-20250428164502.png)

![](images/Set%20集合/file-20250428164515.png)

>事实如此，当输出时会调用父类的 `toString` 方法，里面有调用迭代器的代码，所以 `KeySet` 对象更像是一种特殊的引用（占用十分小的内存），告诉使用者这些元素存放在哪，但实际底层中是一种中介，动态的调用迭代器来获取元素

****
### 1.1.2 值 Value

>每个键只对应一个值，但是不同键可以指向相同的值，所以 `Value` 才可以看作是集合

![](images/Set%20集合/file-20250428144016.png)

>这个方法的整体操作和上面的 `KeySet` 类似，也是作为一个视图，可以直接通过 `Values` 对象删除值（整个键值对删除），但是不能修改或增加，这需要通过 `Map` 对象来操作

> `Values` 对象也是同理，只不过继承的父类不同而已

****
## 2. 迭代方式

### 2.1 迭代器 + while

```java
Iterator<Integer> iterator = keys.iterator();  
while (iterator.hasNext()) {  
    Integer key = iterator.next();  
    String value = map.get(key);  
    System.out.println(key + "=" + value);  
}
```

>在 `HashMap` 中也存在迭代器，通过迭代器来获取 `Set` 集合中的所有键，然后再根据键获取对应的值

****
### 2.2 迭代器 + 增强 for

```java
for (Integer key : keys) {  
    String value = map.get(key);  
    System.out.println(key + "=" + value);  
}
```

>这种遍历方式虽然没有直接调用迭代器，但是它的底层会自动调用，把迭代器获取的 `next` 赋值给 `key` ，但是如果集合内部是不带构造器的就不能这样使用了

****
### 2.3 `Map.Entry` 接口

>`Map.Entry` 是用来表示 `Map` 中“一个键值对”的对象

```java
Set<Map.Entry<Integer, String>> entries = map.entrySet();  
Iterator<Map.Entry<Integer, String>> iterator = entries.iterator();  
while (iterator.hasNext()) {  
    Map.Entry<Integer, String> entry = iterator.next();  
    System.out.println(entry.getKey() + "=" + entry.getValue());  
}
```

>把 `Map` 中的键值对封装成一个对象，让他们实现简单的统一，一个对象就代表着一条数据，当需要修改数据时直接通过对象修改，降低了键值对与 `Map` 直接的耦合度，让管理变得更加方便，因为每次 `new` 对象都是不一样的，所有也很符合 `Set` 集合的特性，就可以直接把 `Entry` 对象放进去，并且封装的属性获取更方便，不管拿到的是 `key` 还是 `value`，它们一定是互相对应的，所以就可以实现单独的修改值，那么 `Entry` 类里面就可以封装很多的方法提供给外部使用

>`Entry` 只允许修改 `value`，不允许修改 `key`，这是为了保证 `Map` 的结构完整性和查找正确性。要想修改 `key`，正确做法是：移除旧的 `Entry`，插入新的 `Entry`

****
## 3. Map 的存储结构

![](images/Set%20集合/file-20250428160648.png)

>看图，在调用 `put(key, value)` 方法时，底层会创建一个实现了 `Map.Entry` 接口的 `Node` 类的对象，这个 `Node` 对象其实就可以看作是 `Entry` 对象，每个 `Node` 保存了 `key`、`value`、`hash` 值，还有一个 `next` 指向下一个节点（链表结构），所以只要往 `Map` 中 `put` 新元素，就会创建新的 `Entry` 对象，用来管理键值对，

>所以在 `Map` 的实现中，所有的键值对都直接存储在 `Entry`（或其派生类，比如 `Node`）对象中。
>也就是说，当调用 `put(key, value)` 时，Map 并不会直接存储 `key` 和 `value`，而是创建一个`Entry<K, V>`（或者 `Node<K, V>`）对象，将 `key` 和 `value` 包裹在其中，这个对象会被存入 `Map` 内部的数据结构

>`keySet()` 和 `values()` 这两个方法并不会直接存储键值对，它们只是提供了一种便捷的视图，用来遍历 `Map` 中的所有键或值

****
## 4. 如何通过 Entry 找到键值对

![](images/Set%20集合/file-20250428165452.png)

>可以看到， `HashMap` 的底层定义了一个 `Node<K, V>[]` 数组，这个就是 `Entry` 数组，

![](images/Set%20集合/file-20250428170009.png)

![](images/Set%20集合/file-20250428165911.png)

>然后根据 `key` 的值来计算 `hash` 值，根据 `hash` 值找到对应的 `Entry` 对象大概在哪个位置，然后对比 `key` 的值，返回 `Entry` 对象中的 `value` 字段

>虽然 `Entry` 中的键和值永远是一一对应的，但是 `Map` 想要找到确切的 `value` 还是要根据 `value` 的对比，如果随意修改了 `key` 的值，就会导致 `key.equals(k)` 返回一个 `false` ，也就是找不到确切的存放 `value` 的那个 `Entry` 对象，这也是为什么 `Entry` 类中唯独没有声明修改 `key` 的方法

****
# 2. HashMap

## 2.1 key 的唯一性

```java
public class Person {  
    private String name;  
    private int age;  
  
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    @Override  
    public String toString() {  
        return "Person{" +  
                "name='" + name + '\'' +  
                ", age=" + age +  
                '}';  
    }  
  
    @Override  
    public boolean equals(Object o) {  
        if (this == o) return true;  
        if (o == null || getClass() != o.getClass()) return false;  
        Person person = (Person) o;  
        return age == person.age && Objects.equals(name, person.name);  
    }  
  
}
```

```java
Person person1 = new Person("张三", 20);  
Person person2 = new Person("李四", 34);  
Person person3 = new Person("王五", 40);  
Person person4 = new Person("张三", 20);  
  
Map<Person, String> map = new HashMap<>();  
map.put(person1, "a");  
map.put(person2, "b");  
map.put(person3, "c");  
map.put(person4, "d");  
System.out.println(map);
// {Person{name='王五', age=40}=c, Person{name='张三', age=20}=a, Person{name='李四', age=34}=b, Person{name='张三', age=20}=d}
```

> `Map` 集合中的键是不允许存在一样的，否则会出现覆盖的情况，但是在自定义 `key` 的类型时重写了 `equals` 方法，但还是出现了 `key` “相同” 但没覆盖的情况，证明 `Map` 底层判断 `key` 是否相同并不是单纯的使用 `equals` 方法判断里面的内容，而是还使用了别的东西作为判断条件

****
## 2.2 哈希表的存储原理

![](images/Set%20集合/file-20250428195700.png)

>哈希表（`Hash table`，也叫散列表）， 是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表

>哈希表 `hash table(key，value)`  的做法其实很简单，就是把 `Key` 通过一个固定的算法函数也就是所谓的哈希函数转换成一个整型数字，然后就将该数字对数组长度进行取余，取余结果就当作数组的下标，将 `value` 存储在以该数字为下标的数组空间里，当多个哈希值对数组长度取模结果相同时，就以链表或者树的形式存放在同一个下标数组中，期间需要进行对比 `key` 的值，如果相同就进行覆盖，因为采用链表的方式，在对比时需要一一遍历，所以 Java8 后采取的尾插法更适合于解决哈希冲突

>因为结合了数组与链表的有点，所以哈希表最大的优点就是把数据的存储和查找消耗的时间大大降低，几乎可以看成是常数时间（利用数学公式查找数组下标），而代价仅仅是消耗比较多的内存

****
### 2.2.1 底层代码实现

![](images/Set%20集合/file-20250428202835.png)

![](images/Set%20集合/file-20250428203501.png)

>在调用 `put()` 方法时底层会创建哈希表数组，然后给要插入的值创建 `Noed<K, V>` 对象，封装 `hash` 、 `key` 、 `value` 、 `next` 字段，然后根据 `hash` 值来计算放在哈希表数组的哪个下标处，

![](images/Set%20集合/file-20250428203434.png)

>如果发生哈希碰撞就判断 `equals` 和 `hashCode` 是否都为 true，然后选择插入还是覆盖

![](images/Set%20集合/file-20250428204056.png)

>这里产生了 `key` 相等的情况 ， `e = p` 把当前的节点赋值给一个 e 变量，后面会根据这个 e 是否为空来判断是否出现 `key` 相等的情况，

![](images/Set%20集合/file-20250428204626.png)

>然后就会进入这里，进行覆盖操作（只覆盖 `value` ）

****
### 2.2.2 哈希表的扩容

![](images/Set%20集合/file-20250428204907.png)

>每插入一个新的键值对 `size` 就加一，当键值对的个数超过一定的数量时就会对哈希表进行扩容，当键值对越来越多而哈希表太小的话就非常容易造成哈希冲突，每次发生冲突都需要遍历一遍链表，这样非常浪费时间，所以为了减少冲突就需要扩容机制，让键值对放均匀的放进每个哈希表的下标中

****
### 2.2.3 哈希函数

>哈希函数也叫散列函数，就是把“任意大小的输入”（比如一个对象、字符串），映射成一个“固定范围的整数”，用这个整数作为定位的工具

![](images/Set%20集合/file-20250428211101.png)

>可以看到这里有个判断 `key == null`，也就是说 `null` 也是可以作为 `key` 的，并且它的哈希值就是 0，所以它默认存在哈希表的 0 号下标处

>调用 `put` 方法时会通过 `hash(key)` 来计算哈希值， `(h = key.hashCode()) ^ (h >>> 16)` 这段代码就是具体的计算函数，调用对象的 `hashCode()` 方法拿到原始哈希值，然后进行一系列操作得到最终的哈希值，因为我传入的 `key = 2`，所以它的原始哈希值就是 2

![](images/Set%20集合/file-20250428212102.png)

>当传入的 `key` 是 `String` 类型时它的原始哈希值的处理方式就是这样的，所以根据传入的 `key` 为什么类型来获取对应的原始哈希值，然后再使用统一的函数进行处理


****
## 2.3 自定义 key 类型时需要同时重写 equals 和 hashCode 方法

>判断 `key` 是否相等需要进行两步操作：
>1. 先比较哈希值，通过哈希值找到存放键值对的哈希表的下标
>2. 通过 `equals` 比较内容，当两者全部相同时才判断为 `key` 相同

**如果只重写 hashCode()**

>通过 `key` 的哈希值可能可以定位到确切的哈希表的下标位置，但是比较 `equals` 时会出现错误，导致具有相同的哈希值的数据实际内容却不同，那么底层就会把它们当作不同的 `key`，这就会造成逻辑上它们是相同的 `key`，但是实际上哈希表中出现了很多重复的 `key`，这不仅不符合 `set` 集合的特性，更会造成数据的混乱，一个 `key` 可能返回多个 `value`，同样只重写 `equals` 是一样的，所以自定义 `key` 时创建的对象编译器会要求重写 `equals` 和 `hashCode`

![](images/Set%20集合/file-20250428213554.png)

>当然重写的字段是根据使用者来决定的，是根据名字判断为同一个 `key` 还是全部的字段都相同才判断为同一个 `key` ，这需要手动设置

****
## 2.4 HashMap 的简单实现

>手写简易版的 `HashMap`

```java
public class MyHashMap<K, V> {  
    private Node<K, V>[] table; // 哈希表  
    private int size; // 键值对个数  
  
    static class Node<K, V> {  
        int hash; // 哈希值  
        K key; // 键  
        V value; // 值  
        Node<K, V> next; // 下一个节点  
  
        public Node() {}  
  
        public Node(int hash, K key, V value, Node<K, V> next) {  
            this.hash = hash;  
            this.key = key;  
            this.value = value;  
            this.next = next;  
        }  
  
        @Override  
        public String toString() {  
            return "[" + key + "=" + value + "]";  
        }  
    }  
  
    @SuppressWarnings("unchecked")  
    public MyHashMap() {  
        this.table = new Node[16];  
    }  
  
  
  
    // put 方法  
    public V put(K key, V value) {  
        if (size == table.length) {  
            grow();  
        }  
        if (key == null) {  
            return putForNullKey(value);  
        } else { // 当 key 不为空  
            // 获取 key 的 hash 值  
            int hash = key.hashCode();  
            // 通过 hash 值获取对应的哈希表的下标  
            int index = hash % table.length;  
            // 根据下标获取链表表头的地址  
            Node<K, V> node = table[index];  
            Node<K, V> prev = null;  
            while (node != null) {  
                if (key.equals(node.key)) {  
                    // 找到相同 key 的节点，覆盖 value                    
                    V oldValue = node.value;  
                    node.value = value;  
                    return oldValue;  
                } else {  
                    // 目前没找到相同的 key ，依次遍历链表  
                    prev = node;  
                    node = node.next; // node 依次后移  
                }  
            }  
            // 遍历完整个链表没有找到相同的 key ，直接插到链表尾部  
            if (prev != null) {  
                prev.next = new Node<>(hash, key, value, null);  
            } else {  
                // 遍历完链表后 prev 还是 null 证明整个链表为空，直接插入节点  
                table[index] = new Node<>(hash, key, value, null);  
            }  
            size++;  
        }  
        return null;  
    }  
  
    private V putForNullKey(V value) {  
        // 因为传入的 key 是 null，所以它一定是放在 0 号位置的  
        // 获取 0 号位置的地址引用，得到整个链表的表头  
        Node<K, V> node = table[0];  
        if (node == null) {  
            // 如果链表表头是 null，证明此链表为空  
            // 直接插入就行  
            table[0] = new Node<>(0, null, value, null); // 让新插入的作为链表表头  
            size++; // 插入了一个 key 为 null 的键值对  
            return value;  
        }  
        // 走到这里证明链表中是有节点的，这时候就需要遍历链表查找是否有相同 key 的键值对  
        Node<K, V> prev = null;  
        while (node != null) {  
            if (node.key == null) {  
                // 找到相同 key 的节点，直接覆盖 value                
                V oldValue = node.value;  
                node.value = value;  
                return oldValue; // 返回旧的 value            
            } else {  
                // 没找到相同的 key 的键值对，慢慢遍历  
                prev = node;  
                node = node.next; // node 指针依次后移  
            }  
        }  
        // 遍历完链表都没有找到相同 key 的键值对，那就尾插法插到链表的尾部  
        prev.next = new Node<>(0, null, value, null);  
        size++;  
        return value;  
    }  
  
    // get方法  
    public V get(K key) {  
        int hash = key.hashCode();  
        int index = hash % table.length;  
        Node<K, V> node = table[index];  
        while (node != null) {  
            if (key.equals(node.key)) {  
                return node.value;  
            } else {  
                node = node.next;  
            }  
        }  
        return null;  
    }  
  
    // 扩容  
    public void grow() {  
        int newTableLength = table.length * 2;  
        Node<K, V>[] newTable = new Node[newTableLength];  
        for (int i = 0; i < table.length; i++) { // 遍历老哈希表中的每个位置的链表表头  
            Node<K, V> node = table[i]; // 获取老哈希表中的所有链表表头  
            // 遍历链表中的节点，计算每个节点的新的 hash ，然后放到对应的新的哈希表的下标位置  
            while (node != null) {  
                int index = node.hash % newTableLength; // 获取旧的节点在新的哈希表中的新的 hash 值  
                // 断开当前节点的 next 引用，避免连接到旧链表  
                Node<K, V> nextNode = node.next;  
                node.next = null; // 清理老节点的 next 引用，避免在新的哈希表中某个链表节点的 next 会指向其他链表的节点  
                if (newTable[index] == null) {  
                    newTable[index] = node;  
                } else {  
                    Node<K, V> tail = newTable[index];  
                    while (tail.next != null) { // 遍历到新哈希表中的链表的表尾  
                        tail = tail.next;  
                    }  
                    // 遍历到最后一个节点  
                    tail.next = node; // 将老哈希表的链表的表头插到表尾  
                }  
                node = nextNode; // 继续处理老链表的下一个节点  
            }  
        }  
        table = newTable;  
    }  
  
    public int size() {  
        return size;  
    }  
  
    // 重写 toString 方法，输出 map 时会自动调用  
    @Override  
    public String toString() {  
        StringBuilder sb = new StringBuilder();  
        for (int i = 0; i < table.length; i++) { // 遍历哈希表  
            if (table[i] != null) {  
                Node<K, V> node = table[i];  
                while (node != null) { // 遍历存储键值对的链表  
                    sb.append(node);  
                    sb.append("\n");  
                    node = node.next;  
                }  
            }  
        }  
        return sb.toString();  
    }  
}
```

****
## 2.5 HashMap的容量一直是2的次幂

![](images/Set%20集合/file-20250429154407.png)

>这段代码就是底层用来控制底层的哈希表的容量始终是 2 的次幂

![](images/Set%20集合/file-20250429155613.png)

>可以看到

****

# 4. LinkedHashMap

