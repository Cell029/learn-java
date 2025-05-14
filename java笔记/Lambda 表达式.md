
# 1. Lambda 表达式

>Lambda 表达式是用来代替只有一个方法的接口的匿名实现类写法，使得代码更简洁、更易读

```
(参数列表) -> { 代码逻辑 }
```

```java
// 无参数，无返回值 
() -> log.info("Lambda") 
// 有参数，有返回值 
(int a, int b) -> { a+b }

等价于

log.info("Lambda");
private int plus(int a, int b){ 
	return a+b; 
}
```

>最常见的 Lambda 表达式的使用就是新建线程，`new Thread` 需要一个实现 `Runnable` 接口类型的对象实例作为参数，使用匿名内部类 + Lambda 表达式的方式就不用找对象接收，直接把它当做参数

```java
new Thread(new Runnable() { 
	@Override 
	public void run() { 
		System.out.println("快速新建并启动一个线程"); 
	} 
}).start();

// 等价于

new Thread(()->{ // 因为 run 方法不用传参，所以可以直接使用 ()
	System.out.println("快速新建并启动一个线程"); 
}).start();
```

>由此可见 Lambda 实际上是 JVM 优化后的匿名内部类的一种形式

****
# 2. 函数式接口

## 2.1 定义

>函数式接口要求接口只有一个抽象方法，但它可以有多个默认方法或静态方法，可以通过使用 `@FunctionalInterface` 注解来标明该接口是一个函数式接口。

```java
@FunctionalInterface
public interface MyFunction {
    // 一个抽象方法
    void apply();

    // 可选的：可以有多个默认方法
    default void defaultMethod() {
        System.out.println("This is a default method.");
    }

    // 可选的：可以有静态方法
    static void staticMethod() {
        System.out.println("This is a static method.");
    }
}
```

```java
@FunctionalInterface
public interface MyFunction {
    void apply();
}

public class Test {
    public static void main(String[] args) {
        // 使用Lambda表达式实现接口
        MyFunction myFunction = () -> System.out.println("Hello, World!");

        // 调用apply方法
        myFunction.apply();  // 输出：Hello, World!
    }
}
```

****
## 2.2 函数式思想

>Lambda 更大的作用是在解决具体的问题上，并非单纯创造一个匿名类

```java
public List<Goods> filterByType(List<Goods> list, String type) {
    List<Goods> result = new ArrayList<>();
    for (Goods goods : list) {
        if (goods.getType().equals(type)) {
            result.add(goods);
        }
    }
    return result;
}
```

>如果定义了一个对商品类型进行排查的方法，它可能是这样的，但是如果需要重新定义一个新的方法用来对金额进行排查，就得重新写一串代码

```java
public List<Goods> filterByPrice(List<Goods> list, Integer price) {
    List<Goods> result = new ArrayList<>();
    for (Goods goods : list) {
        if (goods.getPrice() > price) {
            result.add(goods);
        }
    }
    return result;
}
```

>但实际上除了 `if` 语句里的内容不一样，其余的代码基本一致，这就造成了一旦筛选的条件发生改变，需要更改的代码就非常的多，这对于未来的代码维护是十分不利的，但如果将行为参数化，用函数参数代替判断逻辑，就可以以函数的形式将判断逻辑传入方法中，未来只需要对判断逻辑进行维护就行

```java
public List<Goods> filter(List<Goods> list, Predicate<Goods> predicate) {
    List<Goods> result = new ArrayList<>();
    for (Goods goods : list) {
        if (predicate.test(goods)) {
            result.add(goods);
        }
    }
    return result;
}
```

>而 Lambda 表达式则可以这样写，

```java
List<Goods> goodsList = ...;

// 按类型筛选
List<Goods> fruits = filter(goodsList, g -> g.getType().equals("水果"));

// 按价格筛选
List<Goods> expensive = filter(goodsList, g -> g.getPrice() > 100);
```

****
## 2.3 Java 内置的四种函数式接口

### 1. Consumer

>它是一个消费函数式接口，主要针对的是“消费”这个场景，它接收一个参数，但不返回结果，常用于执行某个动作，如打印、写入文件等

![](images/Lambda%20表达式/file-20250514153012.png)

```java
Consumer<String> printer = new Consumer<String>() {  
    @Override  
    public void accept(String s) {  
        System.out.println("打印:" + s);  
    }  
};  
printer.accept("hello");

// 等价于

Consumer<String> printer = s -> System.out.println("打印:" + s);
printer.accept("hello");

// 等价于
Consumer<String> printer = System.out::println;
printer.accept("hello");
```

>`Consumer` 接口中定义了一个默认方法 `andThen`，它允许多个 `Consumer` 顺序执行

```java
Consumer<String> first = s -> System.out.println("第一步处理：" + s);
Consumer<String> second = s -> System.out.println("第二步处理：" + s);

Consumer<String> combined = first.andThen(second);

List<String> dataList = Arrays.asList("数据A", "数据B", "数据C");
for (String data : dataList) {
    combined.accept(data); // 对每个数据执行 first → second
}
```

```
第一步处理：数据A
第二步处理：数据A
第一步处理：数据B
第二步处理：数据B
第一步处理：数据C
第二步处理：数据C
```

****
### 2. Supplier

>它主要针对的是`get`这个场景或者说获取这个场景，它不接受参数但返回一个结果，常用于延迟提供或生成数据

![](images/Lambda%20表达式/file-20250514155551.png)

```java
Supplier<String> supplier = new Supplier<String>() {  
    @Override  
    public String get() {  
        return "hello";  
    }  
};  
System.out.println(supplier.get()); // hello

// 等价于

Supplier<String> supplier = () -> "hello";
System.out.println(supplier.get());
``` 

>`Supplier` 适合与构造方法或静态方法的方法引用（`::`）结合使用

```java
class Product {
    public Product() {
        System.out.println("创建 Product 对象");
    }
}

public class ConstructorReference {
    public static void main(String[] args) {
        Supplier<Product> supplier = Product::new;
        Product p = supplier.get();
    }
}
```

**延迟执行：**

```java
public static void main(String[] args) {
    printIfDebug(() -> expensiveCalculation()); 
    // Supplier<String> supplier = () -> expensiveCalculation()
    // printIfDebug(supplier)
}

static void printIfDebug(Supplier<String> supplier) {
    boolean debug = false;
    if (debug) {
        System.out.println(supplier.get());
    }
}

static String expensiveCalculation() {
    System.out.println("执行复杂计算");
    return "计算结果";
}
```


>由于 `debug` 是 `false`，所以无法执行 `System.out.println(supplier.get())` ，而不调用 `supplier.get()` 就不会调用 `expensiveCalculation()` ，所以整个 `expensiveCalculation()` 从未被执行过，这就是 `Supplier` 实现的延迟计算

****
### 3. Predicate

>**Predicate** 抽象了判断这个场景，而且这种抽象不局限于业务，是直接对某一类场景进行抽象

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

```java
Predicate<String> predicate = new Predicate<String>() {  
    @Override  
    public boolean test(String s) {  
        return s.length()  >5;  
    }  
};  
System.out.println(predicate.test("hello")); // false  
System.out.println(predicate.test("hello world")); // true

// 等价于

Predicate<String> predicate = s -> s.length > 5;
System.out.println(isLongEnough.test("hello")); // false
System.out.println(isLongEnough.test("helloworld")); // true
```

****
### 4. Function

>它定义了一个 `apply()` 方法，用于执行将输入类型 `T` 转换为输出类型 `R` 的操作

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

>将字符串类型转换成一个 int 类型

```java
Function<String, Integer> function = new Function<String, Integer>() {  
    @Override  
    public Integer apply(String s) {  
        return s.length();  
    }  
};  
System.out.println(function.apply("hello"));

// 等价于

Function<String, Integer> function = s -> s.length();
System.out.println(function.apply("hello"));
```



****
# 3. Lambda 常见格式

1、无参数，无返回值

```java
() -> {
    System.out.println("Hello Lambda");
};
```

2、一个参数，无返回值

```java
(x) -> {
    System.out.println(x);
};
```

3、多个参数，有返回值

```java
(a, b) -> {
    return a + b;
};
```

4、简化写法（仅一行代码可省略花括号、return 和类型）

```java
(a, b) -> a + b;
```

>如果方法的参数列表中的参数数量有且只有⼀个，参数列表的小括号是可以省略不写的（省略掉小括号的同时， 必须要省略参数的类型）；当⼀个方法体中的逻辑有且只有⼀句的情况下，⼤括号可以省略；如果⼀个方法中唯⼀的⼀条语句是⼀个返回语句， 此时在省略掉大括号的同时， 也必须省略掉 return

5、重要特征

- **可选类型声明：** 不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：** 一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：** 如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：** 如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

****
# 4. 与匿名内部类的关系

>Lambda 表达式和匿名内部类在功能上是相似的，二者都可以用来实现接口或类中的抽象方法，但Lambda 表达式可以看作是匿名内部类的一种简化形式，它让代码更加简洁，特别是在接口中只有单个方法时

**主要区别：**

1、Lambda 表达式依赖于目标类型（通常是一个函数式接口）来推断参数类型和返回类型，而匿名内部类必须显式实现接口或继承类，因此编译器需要更多的上下文来确定类型

2、Lambda 表达式并不会真实的创建一个 .class 文件，而匿名内部类每次创建时都会生成一个新的 .class 文件

**注意：**

>如果在`lambda表达式`中，使用到了局部变量，那么这个局部变量会被隐式的声明为 final 是⼀个常量，不能修改值（与匿名内部类一致）

```java
public static void main(String[] args) {
	int num = 10; // 注意：这个变量被 lambda 捕获了

    Runnable r = () -> {
        System.out.println("num = " + num); // Lambda 中使用 num
    };
    // num = 20; // 编译报错
    r.run();
}
```

****
# 5. 方法引用

## 5.1 实例方法的引用

```
方法归属者::方法名 // 静态方法的归属者为类名，普通方法归属者为对象
```

>在引用的方法后⾯，不要添加小括号，并且引用的这个方法， 参数（数量、类型） 和 返回值， 必须要跟接口中定义的⼀致

```java
PrintStream out = System.out;
Consumer<String> printer = out::println;
printer.accept("Hello"); // 输出：Hello

// 等价于

Consumer<String> printer = s -> out.println(s);
```

****
## 5.2 静态方法的引用

```java
Function<String, Integer> func = Integer::parseInt;
System.out.println(func.apply("123")); // 输出：123

// 等价于

Function<String, Integer> func = s -> Integer.parseInt(s);
```

****
## 5.3 特殊方法的引用

```
类名::实例方法
```

>这种形式其实是将“Lambda 的第一个参数”作为“调用者”，第二个参数作为方法参数

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");
list.sort(String::compareToIgnoreCase);

// 等价于

list.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
```

>需要注意的是：`::` 前的类名指的是调用方法的那个参数的类型所属的类（不是方法返回的类型），因为 `s1` 是字符串，所以使用的是 `String` 

```java
Function<Vip, String> function = new Function<Vip, String>() {
    @Override
    public String apply(Vip vip) {
        return vip.getName();
    }
};

Function<Vip, String> function = Vip::getName;

// 等价于
Function<Vip, String> function = vip -> vip.getName();
```

>用传进的参数作为对象使用，引用的是任意对象的实例方法，所以不能直接用具体的对象使用，因为传进的参数是 `Vip` 类的实例，所以使用的类名是 `Vip` 

****
## 5.4 构造方法的引用

```
类名::new
```

1、无参构造

```java
Supplier<Student> supplier = Student::new;
// 等价于：
Supplier<Student> supplier = () -> new Student();
```

2、一个参数的构造方法

```java
Function<String, Student> function = Student::new;
// 等价于：
Function<String, Student> function = name -> new Student(name);
```

3、两个参数的构造方法

```java
BiFunction<String, Integer, Student> biFunction = Student::new;
// 等价于：
BiFunction<String, Integer, Student> biFunction = (name, age) -> new Student(name, age);
```

****
## 5.5 数组的引用

```
类型[]::new
```

```java
Function<Integer, int[]> arrayCreator = int[]::new;
String[] arr = arrayCreator.apply(5);

// 等价于
Function<Integer, int[]> arrayCreator = num -> new int[num]
```

****
# 6. 集合中的应用

## 6.1 集合的遍历

![](images/Lambda%20表达式/file-20250514165834.png)

>集合的 `forEach()` 需要的参数类型是 `Consumer` ，所以可以使用 Lambda 表达式

```java
ArrayList<Integer> list = new ArrayList<>();  
Collections.addAll(list, 1,2,3,4,5);
list.forEach(new Consumer<Integer>() {  
    @Override  
    public void accept(Integer integer) {  
        System.out.println(integer);  
    }  
});

// 等价于
list.forEach(i -> System.out.println(i));

// 等价于
list.forEach(System.out::println);
```

```java
Map<String, Integer> map = new HashMap<>();  
map.put("A", 1);  
map.put("B", 2);  
map.put("C", 3);  
  
map.forEach(new BiConsumer<String, Integer>() {  
    @Override  
    public void accept(String s, Integer integer) {  
        System.out.println("key:" + s + ",value:" + integer);  
    }  
});  
  
// 等价于
  
map.forEach((k, v) -> System.out.println("key:" + k + ",value:" + v));
```

****
## 6.2 删除集合中的元素

![](images/Lambda%20表达式/file-20250514171945.png)

> `removeIf()` 需要的参数类型是 `Predicate` ，所以可以使用 Lambda 表达式

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));  
System.out.println(list);  
list.removeIf(new Predicate<String>() {  
    @Override  
    public boolean test(String s) {  
        return s.equals("C");  
    }  
});  

// 等价于
list.removeIf(s -> s.equals("C"));

// 等价于
list.removeIf("C"::equals);

System.out.println(list);
```

****
## 6.3 集合内的排序

![](images/Lambda%20表达式/file-20250514173437.png)

>通过 Lambda 表达式来重写匿名内部类中的 `compareTo()` 方法

```java
List<String> list = Arrays.asList("banana", "apple", "orange");  
  
Collections.sort(list, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return s1.compareTo(s2);  
    }  
});  

// 等价于
Collections.sort(list, (s1, s2) -> s1.compareTo(s2));

// 等价于
Collections.sort(list, String::compareTo);


System.out.println(list);
```

****
