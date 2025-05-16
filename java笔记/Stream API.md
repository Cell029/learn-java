
# 1. 概述

>Java 流（Stream）是一连串的元素序列，元素一个一个地经过加工与处理（串行处理），流式编程的概念基于函数式编程的思想，每个加工步骤就是一个函数式操作（如 `map`, `filter`, `sorted` 等），它不改变原集合而是产生一个新的结果，但可以简化代码，提高可读性和可维护性
>
>流并不会存储数据，它更像是一种封装好的工具，数据在流中被流动处理，每次使用流只能完成一次遍历，操作完成后立即失效，再次使用需要重新创建新的流

Stream 流默认是串行的，因为串行更容易控制，结果的输出顺序一致、无并发干扰，行为是确定的，对于绝大多数场景来说这样更安全、更稳定。因为大多数集合类（如 `ArrayList`）本身不是线程安全的，如果默认并发执行可能会引发线程安全问题，不过可以通过使用 `.parallelStream()` 方法让 Stream 成为并行流

# 2. Stream 流的 3 种操作类型

- 创建 Stream
- Stream 中间处理
- 终止 Steam

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

#### 2.1 map

>将流中每个元素映射成另外一个元素，形成一个新的流，但原始流不变

![](images/Stream%20API/file-20250515133125.png)

>`map` 里面要求的参数是一个 `Function` 接口，而这个接口就是用来转换类型的

```java
List<String> list = Arrays.asList("1", "2", "3");  
List<Integer> intList = list.stream()  
        .map(new Function<String, Integer>() {  
            @Override  
            public Integer apply(String s) {  
                return Integer.parseInt(s);  
            }  
        })  
        .collect(Collectors.toList());  
System.out.println(intList); // [1, 2, 3]
```

```java
// 等价于
List<Integer> intList = list.stream()  
        .map(Integer::parseInt)  
        .collect(Collectors.toList()); 
```

```java
class User {
    private String name;
    private int age;
    public User(String name, int age) { this.name = name; this.age = age; }
    public String getName() { return name; }
    public int getAge() { return age; }
}

List<User> users = Arrays.asList(new User("Alice", 20), new User("Bob", 22));
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());
System.out.println(names); // [Alice, Bob]
```

****

**1、`mapToInt(ToIntFunction<? super T> mapper)`**

>将一个对象流（如 `Stream<String>`）中的元素转换为 `int` 类型，得到一个 `IntStream`，以此避免数据的自动装箱与拆箱

```java
List<String> list = Arrays.asList("1", "2", "3");
IntStream intStream = list.stream()
        .mapToInt(s -> Integer.parseInt(s)); // 转为 IntStream
int sum = intStream.sum(); // IntStream 支持 sum, average 等操作
System.out.println(sum); // 6
```

**2、`mapToDouble(ToDoubleFunction<? super T> mapper)`**

>将对象流映射为 `double` 类型，返回一个 `DoubleStream`

```java
List<String> prices = Arrays.asList("19.9", "5.5", "10.0");
DoubleStream priceStream = prices.stream()
        .mapToDouble(s -> Double.parseDouble(s)); // 转为 DoubleStream
double average = priceStream.average().orElse(0.0);
System.out.println(average); // 输出平均值：11.8
```

**3、`mapToLong(ToLongFunction<? super T> mapper)`**

>将对象流映射为 `long` 类型，返回一个 `LongStream`

```java
List<String> values = Arrays.asList("10000000000", "20000000000");
LongStream longStream = values.stream()
        .mapToLong(s -> Long.parseLong(s));
long max = longStream.max().orElse(0L);
System.out.println(max); // 输出 20000000000
```

****
#### 2.2 flatMap

>`flatMap()` 是用来处理嵌套集合、嵌套结构的，先把每个元素处理并返回一个新的Stream，然后将多个 Stream 展开合并为了一个完整的、新的 Stream 

![](images/Stream%20API/file-20250515144947.png)

```java
List<Integer> list1 = new ArrayList<Integer>();  
list1.add(1);  
list1.add(2);  
list1.add(3);  
List<Integer> list2 = new ArrayList<Integer>();  
list2.add(4);  
list2.add(5);  
list2.add(6);  
  
Stream<List<Integer>> listStream = Stream.of(list1, list2);  
listStream.flatMap(new Function<List<Integer>, Stream<?>>() {  
    @Override  
    public Stream<?> apply(List<Integer> integers) {  
        return integers.stream();  
    }  
}).forEach(System.out::print);
```

```
原始数据: [[1, 2, 3], [4, 5, 6]]

map: [
	    Stream(1, 2, 3),
        Stream(4, 5, 6)
     ]

flatMap: Stream(1, 2, 3, 4, 5, 6)
```

>将两个集合转换成一个流输出

```java
List<String> lines = Arrays.asList("hello world", "java stream");
List<String> words = lines.stream()
    .flatMap(line -> Arrays.stream(line.split(" ")))
    .collect(Collectors.toList());
System.out.println(words); // [hello, world, java, stream]
```

```
原始数据: [["hello world"], ["java stream"]]
map: [
		Stream("hello", "world"),
		Stream("java", "stream")
	 ]
flatMap: Stream("hello", "world", "java", "stream")
```

>对象属性的展开

```java
class Student {
    String name;
    List<String> courses;
    public Student(String name, List<String> courses) {
        this.name = name;
        this.courses = courses;
    }
    public List<String> getCourses() { return courses; }
}

List<Student> students = Arrays.asList(
    new Student("Alice", Arrays.asList("Math", "English")),
    new Student("Bob", Arrays.asList("Physics", "Chemistry"))
);

List<String> allCourses = students.stream()
    .flatMap(student -> student.getCourses().stream())
    .distinct()
    .collect(Collectors.toList());
System.out.println(allCourses); // [Math, English, Physics, Chemistry]
```

```
原始数据: [
			["Alice", ["Math", "English"]], 
			["Bob", ["Physics", "Chemistry"]]
		 ]
map: [
		Stream("Math", "English"),
		Stream("Physics", "Chemistry")
	 ]
flatMap: Stream("Math", "English", "Physics", "Chemistry")
```

>所以 `flatMap` 就是在 `map` 的基础上，把每个元素变成的子流统一扁平化为一个流，

****
### 3. filter

![](images/Stream%20API/file-20250515151609.png)

>`filter` 接收一个 `Predicate<T>` 函数式接口，表示对流中元素的布尔条件判断，它用于从原始数据流中筛选出符合条件的元素，是一种行为抽象的方式，常作为中间操作链中的首个步骤，用于提前剔除不必要的数据，从而减少后续操作的处理成本。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evens = numbers.stream()
    .filter(new Predicate<Integer>() {  
	    @Override  
	    public boolean test(Integer integer) {  
	        return integer % 2 == 0;  
	    }  
	}) 
    .collect(Collectors.toList());

System.out.println(evens); // [2, 4, 6]

// 等价于
.filter(integer -> integer % 2 == 0)
```

****
### 4. distinct

>把流中的重复元素过滤掉，只保留元素的第一个出现，保证返回的流中元素唯一

```java
List<Integer> list = Arrays.asList(1, 2, 3, 2, 1, 4, 5);
List<Integer> distinctList = list.stream()
    .distinct()
    .collect(Collectors.toList());

System.out.println(distinctList); // [1, 2, 3, 4, 5]
```

>如果是对自定义的对象使用的话，就需要自定义的类中重写 `equals()` 和 `hashCode()` 方法

```java
class Person {
    String name;
    int age;
    Person(String name, int age) {
        this.name = name; this.age = age;
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person p = (Person) o;
        return age == p.age && Objects.equals(name, p.name);
    }
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    @Override
    public String toString() {
        return name + ":" + age;
    }
}

List<Person> persons = Arrays.asList(
    new Person("Tom", 20),
    new Person("Jerry", 25),
    new Person("Tom", 20)
);

List<Person> distinctPersons = persons.stream()
    .distinct()
    .collect(Collectors.toList());

System.out.println(distinctPersons); // [Tom:20, Jerry:25]
```

****
### 5. sorted

>对基础类型排序

```java
List<Integer> numbers = Arrays.asList(3, 5, 1, 4, 2);
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sorted); // [1, 2, 3, 4, 5]
```

>对自定义对象排序，使用时需要传入比较器（若类的内部没有重写比较器）

```java
class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    public String toString() { return name + " - " + age; }
}

List<Person> people = Arrays.asList(
    new Person("Tom", 25),
    new Person("Alice", 20),
    new Person("Bob", 23)
);

List<Person> sorted = people.stream()
    .sorted(Comparator.comparingInt(Person::getAge))
    .collect(Collectors.toList());

System.out.println(sorted); // [Alice - 20, Bob - 23, Tom - 25]
```

****
### 6. limit 和 skip

> `limit` 用于截断流，限制从流中获取的元素数量

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result = list.stream()
    .limit(3) // 只取前 3 个元素
    .collect(Collectors.toList());
    System.out.println(result); // [1, 2, 3]
```

> `skip` 用于跳过流中的前 n 个元素，返回剩下的元素组成的新流

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eva");

List<String> result = names.stream()
    .skip(2)
    .collect(Collectors.toList());
System.out.println(result) // [Charlie, David, Eva]
```


****
### 7. concat

>用于合并两个流，返回一个包含两个流中所有元素的串行流（按先后顺序合并）

```java
Stream<String> stream1 = Stream.of("A", "B", "C");
Stream<String> stream2 = Stream.of("D", "E");

Stream<String> combinedStream = Stream.concat(stream1, stream2);
combinedStream.forEach(System.out::println); // A B C D E
```

****
### 8. peek

![](images/Stream%20API/file-20250515162203.png)

> `peek()` 接收一个 `Consumer` 接口，用来查看当前调用位置流中的元素状态，但不会修改流

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");  
  
List<String> result = list.stream()  
        .filter(s -> s.length() > 5)  
        .peek(new Consumer<String>() {  
            @Override  
            public void accept(String s) {  
                System.out.println("Filtered value: " + s); 
            }  
        })  
        .map(String::toUpperCase)  
        .peek(s -> System.out.println("Mapped value: " + s)) 
        .collect(Collectors.toList());  
```

```
Filtered value: banana
Mapped value: BANANA
Filtered value: cherry
Mapped value: CHERRY
```

>从 `peek` 的结果可以看出流中的元素是被一个一个处理的，具体过程如下

```
从 list 取出 "apple":
    `filter("apple")` -> 长度 5，不通过 -> 丢弃，后续操作不执行
        
取出 "banana":
    filter("banana") -> 长度 6，符合 -> 继续执行
    peek("banana") -> 打印：Filtered value: banana
    map("banana") -> 转大写 -> "BANANA"
    peek("BANANA") -> 打印：Mapped value: BANANA
    collect("BANANA")`
        
取出 "cherry":
    filter("cherry") -> 长度 6，符合 -> 继续执行
    peek("cherry") -> 打印：Filtered value: cherry
    map("cherry") -> 转大写 -> "CHERRY"
    peek("CHERRY") -> 打印：Mapped value: CHERRY
    collect("CHERRY")
```

****
## 2.3 结束管道

>只有调用结束管道方法时，中间管道中的方法才会被实际触发，并且一个流只能有一个终端操作，调用后流就关闭了，返回的通常是集合、数值、布尔值、`Optional` 对象等，不再是 Stream 对象

### 1. Optional 对象

#### 1.1 定义

>在使用 Stream 进行操作时，有些终止操作（如 `findFirst()`、`findAny()`、`max()`、`min()` 等）可能找不到任何结果，如果这些方法直接返回 null，就必须手动做 null 检查，不然很容易丢出异常，为了可以显式地处理结果可能不存在的情况，就规定这些方法返回的是 `Optional` 类型，而不是直接返回值或 null

```java
List<String> list = new ArrayList<>();
String result = list.get(0); // 抛出 IndexOutOfBoundsException

List<String> list = new ArrayList<>();
Optional<String> result = list.stream().findFirst(); // 不会抛异常
```

>通常会在流的终止操作后面加上一些显示处理的结果方法，以防空值的情况，让代码的可读性更高

```java
Optional<String> result = list.stream().findFirst();
String value = result.orElse("默认值"); // 当 result 为空时就将 result 赋值为 xxx
```

下面是常见返回 `Optional` 的 Stream 终止方法：

|方法名|返回类型|说明|
|---|---|---|
|`findFirst()`|`Optional<T>`|返回第一个元素（可能不存在）|
|`findAny()`|`Optional<T>`|返回任意一个元素（并行流中可能更快）|
|`min()`|`Optional<T>`|返回最小元素（通过 Comparator 比较）|
|`max()`|`Optional<T>`|返回最大元素（通过 Comparator 比较）|
|`reduce()`|`Optional<T>`|如果不提供初始值，可能会没有结果|
****
#### 1.2 常用方法

##### 1. `isPresent()` 、 `isEmpty()` 、`ifPresent()`

- `isPresent()`：返回 `true` 表示值存在，常与 `if` 配合使用
- `isEmpty()`：返回 `true` 表示值不存在，是 `!isPresent()` 的语义替代
- `ifPresent(Consumer<? super T)` ：如果值存在，则执行消费者行为

```java
Optional<String> opt = Optional.of("java");
if (opt.isPresent()) {
    System.out.println(opt.get());
}
```

```java
Optional<String> opt = Optional.of("stream");
opt.ifPresent(s -> System.out.println(s.toUpperCase()));  // 输出：STREAM
```

****
##### 2. `get()`

- 作用：获取 Optional 中的值。

```java
Optional<String> opt = Optional.of("hello");
System.out.println(opt.get());  // 输出：hello
```

>如果此时的 opt 为空，则会抛出异常

****
##### 3. `orElse(T other)` 、 `orElseGet(Supplier<? extends T>)` 、`orElseThrow()`

-  `orElse(T other)` ：有值返回值，无值返回传入的默认值。

```java
Optional<String> opt = Optional.empty();
String result = opt.orElse("default");
System.out.println(result); // default
```

>需要注意的是：即使 opt 不为空，`orElse()` 也会执行传入的参数表达式

```java
public static void main(String[] args) {
    String result = Optional.of("value").orElse(getDefault());
    // 输出: 调用了 getDefault()
    System.out.println(result); // value
}

public static String getDefault() {
    System.out.println("调用了 getDefault()");
    return "default";
}
```

- `orElseGet(Supplier<? extends T>)` ：用法类似 `orElse()`，但 `Supplier` 是惰性求值，只有在 Optional 为空时，才会执行 `Supplier`

```java
String result = Optional.empty().orElseGet(() -> {
    System.out.println("调用了Supplier");
    return "lazy default";
});
```

- `orElseThrow()`：用于强制性地要求值存在的逻辑，如果不存在执行方法体中的抛出异常逻辑

```java
String result = Optional.of("abc").orElseThrow(); // 正常
String str2 = Optional.empty().orElseThrow(() -> new RuntimeException("没有值")); // 抛异常
```

****
##### 4. `map(Function<? super T, ? extends U>)`

![](images/Stream%20API/file-20250515183106.png)

>与 Stream 中的 `map()` 类似，都是传入一个 `Function` 接口，实现类型的转化功能，但它一次最多只能处理一个元素（处理 null 值不报错）

```java
Optional<User> userOpt = Optional.empty(); // 可能为空
Optional<String> name = userOpt.map(User::getName); // 不会抛出异常
```

****
##### 5. `flatMap(Function<? super T, Optional<U>>)`

>适用于链式操作中需要避免 `Optional<Optional<T>>` 嵌套的情况

```java
Optional<String> opt = Optional.of("stream");
Optional<Integer> result = opt.flatMap(s -> Optional.of(s.length()));
System.out.println(result.get());  // 6
```

****
### 2. 终止操作

#### 2.1 forEach

![](images/Stream%20API/file-20250515184756.png)

>对流中的每一个元素执行指定的动作，例如打印、写文件等

```java
List<String> names = Arrays.asList("Alice", "Jack", "Mike");
names.stream().forEach(name -> System.out.println(name));
```

****
#### 2.2 metch

###### 1. anyMatch

>只要有一个匹配的元素就返回 true

```java
List<String> list = new ArrayList<String>();  
list.add("a");  
list.add("b");  
list.add("c");  
boolean has = list.stream().anyMatch(s -> s.equals("b"));  
System.out.println(has); // true
```

##### 2. allMatch

>需要全部匹配才返回 true

```java
boolean has = list.stream().allMatch(s -> !s.isEmpty());  
System.out.println(has); // true
```

##### 3. noneMatch

>当全部不匹配时返回 true

```java
boolean has = list.stream().noneMatch(s -> s.contains("C"));  
System.out.println(has); // true
```

****
#### 2.3 reduce

>一种将流中的多个元素“归约”为一个值的操作，例如把所有数加起来求和、把所有字符串拼接成一个大字符串、找出最大值、最小值等，也就是把一个集合变成单个值

##### 1. `Optional<T> reduce(BinaryOperator<T> accumulator)`

>不传递初始值，最终结果用 `Optional` 包裹

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);  
Optional<Integer> sum = list.stream()  
        .reduce(Integer::sum); // 等价于: (((1 + 2) + 3) + 4) 
sum.ifPresent(System.out::println); // 10
```

##### 2. `T reduce(T identity, BinaryOperator<T> accumulator)`

>提供一个初始值，返回结果的类型是初始值的类型

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4); 
int sum = list.stream()
    .reduce(0, (a, b) -> a + b); // 等价于: ((((0 + 1) + 2) + 3) + 4)
sum.ifPresent(System.out::println); // 10    
```

```java
List<String> strs = Arrays.asList("A", "B", "C");
String result = strs.stream()
    .reduce("", (a, b) -> a + b); // 字符串拼接
System.out.println(result); // ABC
```

****
#### 2.4 归集

>Stream 的收集操作（`collect()`）本质上就是：把流中的元素经过一套“规则”（由收集器定义，例如：对集合中的元素进行统计、字符串拼接、分组等操作）加工处理后，最终“归集”为一个结果容器

##### 1. collect、Collector、Collectors 区别与关联

>1、`collect` 是 Stream 流的一个终止方法，会使用传入的收集器对结果执行相关的操作，这个收集器必须是 Collector 接口的某个具体实现类
>
>2、`Collector` 是一个接口，collect 方法的收集器是 Collector 接口的具体实现类
>
>3、`Collectors` 是一个工具类，提供了很多的静态工厂方法，提供了很多 Collector 接口的具体实现类，是为了方便程序员使用而预置的一些较为通用的收集器（如果不使用 Collectors 类，也可以手动实现 Collector 接口）

****
##### 2. Collector

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();                 // 创建中间容器
    BiConsumer<A, T> accumulator();         // 累加逻辑（元素加入容器）
    BinaryOperator<A> combiner();           // 并行流合并多个容器
    Function<A, R> finisher();              // 容器转为最终结果
    Set<Characteristics> characteristics(); // Collector 的特性
}
```

>Collector 定义了五个接口类型变量，`Supplier<A>` 负责创建临时容器（存放流中的元素）；`BiConsumer<A, T>` 负责告诉 Stream 每个元素如何添加进临时容器；如果是并行流，多个线程会创建多个容器，就会用 `BinaryOperator<A>` 合并容器；`Function<A, R>` 负责将将临时容器（中间结果）转换成最终结果类型；最后让 `Set<Characteristics>` 设置一个特征值，告诉 Stream 元素最终应该怎么放进容器中
###### 2.1 恒等处理

>恒等处理指的就是 Stream 的元素在经过 Collector 函数处理前后完全不变，例如 `toList()` 操作，只是最终将结果从 Stream 中取出放入到集合对象中，并没有对元素本身做任何的更改处理

```java
list.stream().collect(Collectors.toList()); // 具体是哪种 List 是不确定的，随机的
list.stream().collect(Collectors.toSet());
list.stream().collect(Collectors.toCollection(Supplier)); // 可以自己提供容器类型
// 例如
list.stream().collect(Collectors.toCollection(ArrayList::new));
list.stream().collect(Collectors.toCollection(HashSet::new));
```

map 集合的归集

```java
map.stream().collect(Collectors.toMap(对 key 处理, 对 value 处理))
// 例如
Map<String, String> map = Stream.of("1:A", "2:B", "3:C").collect(Collectors.toMap(s -> s.substring(0, s.indexOf(":")), s -> s.substring(s.indexOf(":") + 1)));
```

****
###### 2. 归约汇总

>让 Stream 流中的元素逐个进入 Collector 的处理函数中，处理完后会与上一个元素的处理结果进行合并，然后到一个新的结果，以此类推，直到所有元素处理完，输出最终的结果，和 `reduce` 类似，但因为 `collect()` 可以用于构建任何复杂结构，不局限于简单的数值，所以 `collect()` 更灵活

| 方法                                   | 说明         | 返回类型          |
| ------------------------------------ | ---------- | ------------- |
| `summingInt(ToIntFunction<T>)`       | 求和（int）    | `Integer`     |
| `summingLong(ToLongFunction<T>)`     | 求和（long）   | `Long`        |
| `summingDouble(ToDoubleFunction<T>)` | 求和（double） | `Double`      |
| `counting()`                         | 计数         | `Long`        |
| `maxBy(Comparator<T>)`               | 最大值        | `Optional<T>` |
| `minBy(Comparator<T>)`               | 最小值        | `Optional<T>` |
| `averagingInt(ToIntFunction<T>)`     | 平均值（int）   | `Double`      |
| `joining()`                          | 字符串拼接      | `String`      |

****
###### 3. 分组分区

>`Collectors.groupingBy` 是 Java Stream 中用于分组的收集器，其本质是：根据分组函数的返回值对 Stream 中的元素进行分类，并将相同组的数据放到一起形成一个` Map<K, V>` 

`groupingBy()` 操作需要指定两个关键输入，即分组函数和值收集器：

- 分组函数：这一项定义了如何对元素进行分组，它是一个函数 `Function<T, K>`，输入一个流中的元素 `T`，返回一个用于分类的键 `K`（作为 Map 的 key），如果多个元素在应用此函数后返回的结果相同，那么这些元素就被归到同一组中

```java
class Employee {
    String name;
    String department;
    int salary;
}

Map<String, List<Employee>> map = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

>`Employee::getDepartment` 就是分组函数 `Function<Employee, String>`，如果有多个员工属于 "研发部"，那这个 Map 会有一项："研发部" -> `List<Employee>`（所有部门为“研发部”的员工），即 `Employee::getDepartment` 对应一个 `List<Employee>` 

```java
List<Employee> employees = List.of(
    new Employee("张三", "研发部", 10000),
    new Employee("李四", "销售部", 8000),
    new Employee("王五", "研发部", 12000),
    new Employee("赵六", "销售部", 8500)
);

Map<String, List<Employee>> map = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

```
{
  "研发部" = [张三对象, 王五对象], // key(String 类型) : value(List 类型) 
  "销售部" = [李四对象, 赵六对象]
}
```

- 值收集器：对于分组后的数据元素的进一步处理，所有值收集器都是 Collector 的一种实现，把元素分组后（即 Map 的 key 确定），每组内部的元素怎么处理、怎么变成最终结果的 value，交给值收集器来决定

```java
Map<String, Long> result = 
    employees.stream().collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting() // 值收集器
));
```

>`Collectors.counting()` 是用来统计数量的，所以这个 map 集合的 key 是部门，value 是人数

- 还有一种特殊的分组操作，其分组的 key 类型仅为布尔值，这种情况，也可以通过`Collectors.partitioningBy()` 提供的分区收集器来实现

```java
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 10000));
```

```
{
  true  = [员工1, 员工2, ...], // 工资大于 10000 的
  false = [员工3, 员工4, ...]  // 工资小于等于 10000 的
}
```

>`partitioningBy` 是一种特殊优化过的分组，它不需要像普通的 key 那样动态计算 `hash` 决定存储在哈希表的哪个位置，永远只需要两个位置，一个放 true 一个放 false，每个流元素只需判断一次布尔值，然后直接将其放入对应的 List 中，无需构建新 key、计算 `hash` 或处理冲突，性能开销小于常规的 `groupingBy`

****
# 3. 并行流

## 1. 定义

>并行 Stream 是一种能够利用多核 CPU 优势，对集合元素进行并发处理的工具，它通过将数据拆分为多个子任务，在多个线程中并发处理后合并结果

```java
// 1. Collection 的 stream() 转 parallelStream()
list.parallelStream().forEach(System.out::println);

// 2. Stream 的 parallel() 方法
list.stream().parallel().forEach(System.out::println);
```

>因为是并行处理流中的元素，所以先后处理的顺序是不确定的，如果想要有序输出可以使用 `.forEachOrdered()` 

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
  
System.out.println("=== parallelStream().forEach ===");  
list.parallelStream().forEach(num -> System.out.print(num + " "));  
System.out.println();  
  
System.out.println("=== stream().parallel().forEach ===");  
list.stream().parallel().forEach(num -> System.out.print(num + " "));  
System.out.println();  
  
System.out.println("=== 保证顺序的 forEachOrdered ===");  
list.parallelStream().forEachOrdered(num -> System.out.print(num + " "));  
System.out.println(); // 1 2 3 4 5 6 7 8 9 10 
```

****
## 2. 线程安全问题

>并行流默认情况下所有的并行操作是共享一个线程池的，















