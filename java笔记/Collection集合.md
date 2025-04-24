
![](images/Collection集合/file-20250424165521.png)

## 1. Collection接口

>`Collection` 是一个接口，位于 `java.util` 包中，它是单列集合的根接口，表示一组元素的集合，是接口，不可以直接实例化，必须使用它的子接口或实现类，如 `List`、`Set`

**单列集合**

>只有值没有键的集合

****
## 2. 常用方法

| 方法名                                 | 功能              |
| ----------------------------------- | --------------- |
| `add(E e)`                          | 添加一个元素          |
| `addAll(Collection<? extends E> c)` | 添加一个集合的所有元素     |
| `remove(Object o)`                  | 删除指定元素          |
| `removeAll(Collection<?> c)`        | 删除所有与参数集合中相同的元素 |
| `retainAll(Collection<?> c)`        | 只保留与参数集合中相同的元素  |
| `clear()`                           | 清空集合            |
| `contains(Object o)`                | 判断是否包含某个元素      |
| `containsAll(Collection<?> c)`      | 判断是否包含集合中所有元素   |
| `isEmpty()`                         | 判断集合是否为空        |
| `size()`                            | 返回集合大小          |
| `iterator()`                        | 返回迭代器，用于遍历      |
| `toArray()`                         | 转为 Object 数组    |
| `toArray(T[] a)`                    | 转为指定类型的数组       |

>集合的底层是重写了 `equals` 方法的，而 `remove(Object o)` 和 `contains(Object o)` 在使用时会调用 `equals` 方法的，所以只要传入的对象的内容相等（对象内部已重写 `equals` 方法），就会返回 `true`

****
## 3. Collection的通用迭代

>就是用统一的方式遍历 `Collection` 中的所有元素，而不依赖于集合的具体类型（如是否有索引等）

```java
Iterator it = collection.iterator();
```

>每次调用 `iterator()`，都会返回一个新的 `Iterator` 对象，它负责获取到 `collection` 集合的大小和当前遍历到的位置 `cursor` 以及上一个返回的元素下标 `lastRet`

![](images/Collection集合/file-20250424182009.png)

```java
iterator.hasNext()
```

>判断是否还有元素，它的底层代码会判断当前下标 `cursor` 是否等于集合的大小，不等为 ture 即为有下一个元素，相等证明此时的光标已经指向了集合中最后一个元素的下一个位置，代表已经遍历完集合

![](images/Collection集合/file-20250424182250.png)

```java
iterator.next()
```

>返回当前 `cursor` 指向的元素，返回的是一个数组元素 `elementData[lastRet = i]`

![](images/Collection集合/file-20250424182528.png)

```java
public void test2() {  
    Collection collection = new ArrayList();  
    collection.add("a");  
    collection.add("b");  
    collection.add("c");  
    collection.add("d");  
  
    Iterator iterator = collection.iterator();  
    while (iterator.hasNext()) {  
        System.out.println(iterator.next()); // a b c d  
    }  
}
```

****
## 4. SequencedCollection

>是 Java 21 新增的一个接口，是 `Collection` 接口的子接口，代表一个元素有固定顺序的集合，它同时也是所有有序集合的父接口，在 Java 21 之前，虽然有些集合是“有序的”，但这些类没有统一的接口描述“顺序操作”，所以 Java 21 提出 `SequencedCollection`，为所有有序集合提供统一的顺序操作接口

**常用方法**

**1、`getFirst()` / `getLast()`**

>获取集合中的第一个和最后一个元素

```java
SequencedCollection sequencedCollection = new ArrayList();  
sequencedCollection.add("a");  
sequencedCollection.add("b");  
sequencedCollection.add("c");  
sequencedCollection.add("d");  
System.out.println(sequencedCollection.getFirst()); // a  
System.out.println(sequencedCollection.getLast()); // d
```

****

**2、`addFirst(E e)` / `addLast(E e)`**

>向集合的前面或后面添加元素（双端队列风格）

```java
sequencedCollection.addFirst("start");  
sequencedCollection.addLast("end");  
System.out.println(sequencedCollection); // [start, a, b, c, d, end]
```

****

**3、`removeFirst()` / `removeLast()`**

>从前面或后面删除元素

```java
sequencedCollection.removeFirst();  
sequencedCollection.removeLast();  
System.out.println(sequencedCollection); // [a, b, c, d]
```

****

**4、reversed()**

>`reversed()` 不会修改原集合，而是返回一个反转视图，并不改变原集合

```java
System.out.println(sequencedCollection.reversed()); // [d, c, b, a]
```

****
## 5. 泛型

>泛型是 Java 5 引入的一种机制，允许在类、接口、方法中定义类型参数，用来指定某种类型，而不是在代码里写死具体类型，以此提高代码复用性、提高类型安全性（编译期检查）、避免强制类型转换

```java
List arrayList = new ArrayList();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    System.out.println(item);
}
```

>`ArrayList` 可以存放任意类型，例子中添加了一个 `String` 类型，添加了一个 `Integer` 类型，再使用时都以 `String` 的方式使用，因此程序崩溃了。为了解决类似这样的问题（在编译阶段就可以解决），就引入了泛型

```java
List<String> arrayList = new ArrayList<>();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    System.out.println(item);
}
```

>使用泛型后在编译阶段，编译器就会报错

```java
List list = new ArrayList();
list.add("Hello");
list.add(123);
String str = (String) list.get(1);  // 运行时报错：ClassCastException

// 如果使用泛型就不需要考虑强转的问题，但是在运行前就要检查传入的参数类型
```

****
### 5.1 类型擦除

>Java在编译阶段会进行类型检查和类型推导，但在编译完成后（运行时），泛型信息就会被“擦除”掉，程序运行时就无法知道泛型的真实类型了

```java
List<String> list = new ArrayList<>();
list.add("hello");
String str = list.get(0);
```

>在编译阶段，Java 编译器会检查类型，只能往 `list` 中添加 `String` 类型，`get(0)` 得到的类型会被推断为 `String`，但是到了运行阶段，这个泛型信息被擦除，JVM 实际上看到的是

```java
List list = new ArrayList();  // 泛型擦除后
list.add("hello");            // 还是能运行
String str = (String) list.get(0); // 这里自动加了强制类型转换
```

>所以泛型在编译时只会进行类型检查和自动转换（此时的自动强转也被叫做泛型补偿），但运行时并不不会存在泛型类型的信息

```java
List<String> stringList = new ArrayList<>();  
List<Integer> intList = new ArrayList<>();  
System.out.println(stringList.getClass() == intList.getClass()); // true
```

>不管是 `List<String>` 还是 `List<Integer>`，在运行时其实是同一个类，都只是 `ArrayList` 集合，只不过泛型作为了一种限制手段限制用户的输入而已

>Java 的泛型是在 Java 5 引入的，为了保证与 Java 5 之前版本的字节码和 JVM 保持兼容，Java 才采用类型擦除的设计方案，让泛型信息只在编译阶段生效，在这个阶段，编译器会进行类型检查和类型推导，确保类型安全，运行时，所有的泛型类型信息都会被擦除，替换为原始类型（通常是 `Object` ），并自动插入必要的强制类型转换指令，从而使生成的字节码与老版本 JVM 完全兼容

****
### 5.2 在类上自定义泛型

```java
public class 类名<类型参数1, 类型参数2, ...> {
    // 使用这些类型参数作为类的成员、方法返回值或参数类型等
}
```

>定义一个简单的泛型容器

```java
public class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

```java
Box<String> stringBox = new Box<>();  
stringBox.set("Hello");  
System.out.println(stringBox.get()); // Hello  
  
Box<Integer> intBox = new Box<>();  
intBox.set(123);  
System.out.println(intBox.get()); // 123
```

>定义多个泛型参数的类

```java
public class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

```java
Pair<String, Integer> pair = new Pair<>("age", 18);
System.out.println(pair.getKey() + ": " + pair.getValue()); // age: 18
```

>自定义泛型类的主要作用是增强代码的通用性、可读性与类型安全性，让程序员口语写出不依赖具体类型但依然能编译检查的通用类或方法，这有点类似于实现一个接口，接口只提供方法的声明，不提供实现

>定义的泛型类，就一定要传入泛型类型实参么？并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型

```java
public class Printer<T> {
    public void print(T data) {
        System.out.println(data);
    }
}

new Printer<String>().print("Hello");
new Printer<Integer>().print(123);
new Printer<User>().print(new User());
```

>当使用泛型后，代码的复用性得到了提升，这又有点像多态了，父类型指引用指向子类型对象

****
### 5.3 在方法上自定义泛型

>泛型方法是指在方法定义时引入泛型参数，不依赖于类是否是泛型

```java
public <T> T 方法名(T 参数) {
    // 方法内部可以使用 T 类型
}
```


