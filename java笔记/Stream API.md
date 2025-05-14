
# 1. 概述

>Java 流（Stream）是一连串的元素序列，元素一个一个地经过加工与处理（串行处理），流式编程的概念基于函数式编程的思想，每个加工步骤就是一个函数式操作（如 `map`, `filter`, `sorted` 等），它不改变原集合而是产生一个新的结果，但可以简化代码，提高可读性和可维护性
>
>流并不会存储数据，它更像是一种封装好的工具，数据在流中被流动处理，每次使用流只能完成一次遍历，操作完成后立即失效，再次使用需要重新创建新的流

Stream 流默认是串行的，因为串行更容易控制，结果的输出顺序一致、无并发干扰，行为是确定的，对于绝大多数场景来说这样更安全、更稳定。因为大多数集合类（如 `ArrayList`）本身不是线程安全的，如果默认并发执行可能会引发线程安全问题，不过可以通过使用 `.parallelStream()` 方法让 Stream 成为并行流

# 2. Stream 流的 3 种操作类型

- 创建Stream
- Stream中间处理
- 终止Steam

![](images/Stream%20API/file-20250514201131.png)

>每个 Stream 管道操作类型都包含几个 API 方法，通过这些 API 方法将数据处理好
## 2.1 开始管道

>主要负责新建一个 Stream 流，或者基于现有的数组、List、Set、Map 等集合类型的对象创建出新的 Stream 流

| API              | 功能说明                          |
| ---------------- | ----------------------------- |
| stream()         | 创建出一个新的 stream 串行流对象          |
| parallelStream() | 创建出一个可并行执行的 stream 流对象        |
| Stream.of()      | 通过给定的一系列元素创建一个新的 Stream 串行流对象 |

****
### 1. 从集合创建 Stream

>集合类都提供了 `stream()` 方法

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");  
// 创建顺序流  
Stream<String> stream = list.stream();  
System.out.println(stream); // java.util.stream.ReferencePipeline$Head@b4c966a
```

****
### 2. 从数组创建 Stream

>使用 `Arrays.stream()` 或 `Stream.of()`

```java
int[] nums = {1, 2, 3, 4};  
IntStream intStream = Arrays.stream(nums); // 适用于基本类型  
  
String[] strs = {"A", "B", "C"};  
Stream<String> arrayStream = Arrays.stream(strs); // 对象类型也可以   
  
Stream<String> stream = Stream.of("A", "B", "C");  
```

****
### 3. 从 Map 创建 Stream

>Map 没有 `stream()` 方法，但它的 key（Set 集合）、value（Collection 集合）、entrySet（Set 集合）都实现了 `Collection` 接口，所以可以使用父类的 `.stream()` 创建流

```java
Map<String, Integer> map = new HashMap<>();
map.put("Tom", 90);
map.put("Jerry", 85);
```

**键集合流（keySet）：**

```java
Stream<String> keyStream = map.keySet().stream();
keyStream.forEach(System.out::println); // Tom 和 Jerry
```

>`map.keySet()` 返回一个 `Set<String>`，通过 `.stream()` 创建 `Stream<String>` 对所有键进行处理

**值集合流（values）：**

```java
Stream<Integer> valueStream = map.values().stream();
valueStream.forEach(System.out::println); // 90 和 85
```

>`map.values()` 返回一个 `Collection<Integer>` ，得到 `Stream<Integer>` 可用于对所有 value 进行处理

**键值对流（entrySet）：**

```java
Stream<Map.Entry<String, Integer>> entryStream = map.entrySet().stream();
entryStream.forEach(entry -> System.out.println(entry.getKey() + " = " + entry.getValue()));
```

>因为键值对本身就是用 `Set` 集合存储的，所以 `map.entrySet()` 返回一个 `Set<Map.Entry<K, V>>`，通过 `getKey()` 和 `getValue()` 拿到每个 entry 的 key 和 value

****
### 4. 通过 Stream.builder() 创建

>如果不确定要添加多少个元素到 Stream 中，可以使用 `Stream.builder()` 创建一个 Stream.Builder 对象，并使用其 `add()` 方法来逐个添加元素，最后调用 `build()` 方法生成 Stream 对象

```java
// 创建一个 Stream.Builder<String> 构造器  
Stream.Builder<String> builder = Stream.builder();  
// 使用 add() 方法添加元素  
builder.add("Apple");  
builder.add("Banana");  
builder.add("Cherry");  
// 构建出真正的 Stream
Stream<String> stream = builder.build();  
// 使用流进行遍历或其他操作  
stream.forEach(System.out::println);
```

>相比 `Stream.of(...)` 或 `Arrays.stream(...)` 这种方法不要求一开始就知道所有元素，反而可以配合一些条件判断来动态控制 `add()` 哪些元素，但是得再真正的 Stream 对象创建前就使用完 `add()` 方法，不然后面再使用则会抛出 `IllegalStateException` 异常

****
### 5. 从 IO 资源创建

>Java 8 引入了 `java.nio.file.Files.lines(Path)` 方法，它可以将一个文件中的每一行封装成一个 `Stream<String>`

```java
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
String line;
while ((line = reader.readLine()) != null) {
    // 处理文件的每一行
}
reader.close();
```

>使用了 Stream 流后，本地文件的获取就不需要像 IO 流那样创建一个输入流读取文件，直接将文件地址给 Stream 就可以快速的获取文件内容

```java
Path path = Paths.get("E:\\IOStream\\test05.txt");  
try (Stream<String> stream = Files.lines(path)) {  
    // 使用 stream 逐行处理数据  
    stream.forEach(System.out::println);  
} catch (IOException e) {  
    e.printStackTrace();  
}
```

>因为把文件转换成了 Stream 类型，所以可以调用各种 Stream API，对数据进行懒加载，一行一行地处理，因此不会占用太多的内存

****
## 6. 通过生成器创建

>有时不是处理已有的数据集合，而是希望生成一组重复或递增的数据（如随机数、自然数、斐波那契数列等），这类数据在开始时并不存在于某个集合中，而是通过某种规则动态生成的，Stream 的生成器方法（generate、iterate）正是用来解决这种需求的

1、`Stream.generate()`：通过 Supplier 生成“同质元素”

![](images/Stream%20API/file-20250514221500.png)

>因为 Supplier 不需要传值，并且每调用一次都产生一个新的值，所以生成一些固定的数或者使用一些数学函数生成一些随机数

```java
Stream<Integer> stream = Stream.generate(() -> 0);
stream.limit(5).forEach(System.out::println);
// 输出 0 0 0 0 0
```

```java
Stream<Double> stream = Stream.generate(Math::random);
stream.limit(3).forEach(System.out::println);
// 输出一组 [0,1) 之间的随机数
```

2、`Stream.iterate()`：可迭代递增的流

```java
static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)
```

>`seed` 是初始值（种子），`UnaryOperator<T>` 是一个函数式接口，表示从当前元素计算下一个元素

```java
Stream<Integer> stream = Stream.iterate(0, n -> n + 1);
stream.limit(5).forEach(System.out::println); // 0 1 2 3 4
```

```java
Stream<Integer> stream = Stream.iterate(0, n -> n + 2);
stream.limit(5).forEach(System.out::println); // 0 2 4 6 8
```


****
## 2.2 中间管道

>从创建后到最终收集结果之前，Stream 在中间管道对流中元素进行一系列处理（可叠加），这些步骤不会立即执行，而是等到终止操作触发

| API        | 功能说明                                                            |
| ---------- | --------------------------------------------------------------- |
| filter()   | 按照条件过滤符合要求的元素， 返回新的 stream 流                                    |
| map()      | 将已有元素转换为另一个对象类型，一对一逻辑，返回新的 stream 流                             |
| flatMap()  | 将已有元素转换为另一个对象类型，一对多逻辑，即原来一个元素对象可能会转换为一个或者多个新类型的元素，返回新的 stream 流 |
| limit()    | 仅保留集合前面指定个数的元素，返回新的 stream 流                                    |
| skip()     | 跳过集合前面指定个数的元素，返回新的 stream 流                                     |
| concat()   | 将两个流的数据合并起来为1个新的流，返回新的 stream 流                                 |
| distinct() | 对 Stream 中所有元素进行去重，返回新的 stream 流                                |
| sorted()   | 对 Stream 中所有的元素按照指定规则进行排序，返回新的 stream 流                         |
| peek()     | 对 Stream 流中的每个元素进行逐个遍历处理，返回处理后的 stream 流                        |

### 1. 链式调用

```java
List<String> names = List.of("Tom", "Amy", "Jack", "Jerry");

names.stream()                          // 开始管道
     .filter(name -> name.length() > 3) // 中间操作1
     .map(String::toUpperCase)          // 中间操作2
     .sorted()                          // 中间操作3
     .forEach(System.out::println);     // 终止操作
```

>每调用一个中间操作的方法就会创建一个新的 Stream 对象（这些中间 Stream 对象根本不在堆上分配，或者被立即销毁、回收），但这个对象并不会对之前的数据进行处理，而是将之前进行过的操作记录下来保存好，等到调用结束管道中的方法时再一起对数据进行处理

>这种行为方式就非常适合 JIT 编译器的优化，JIT 可以在最后一次性优化整条操作链，可以消除中间对象的创建（临时对象不逃出方法，不会在堆中分配内存），就像最终只调用了一个方法一样

****
### 2. map 与 flatMap










