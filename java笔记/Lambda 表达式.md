
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

****
# 4. 与匿名内部类的关系

>Lambda 表达式和匿名内部类在功能上是相似的，二者都可以用来实现接口或类中的抽象方法，但Lambda 表达式可以看作是匿名内部类的一种简化形式，它让代码更加简洁，特别是在接口中只有单个方法时

**主要区别：**

1、Lambda 表达式依赖于目标类型（通常是一个函数式接口）来推断参数类型和返回类型，而匿名内部类必须显式实现接口或继承类，因此编译器需要更多的上下文来确定类型

2、Lambda 表达式并不会真实的创建一个 .class 文件，而匿名内部类每次创建时都会生成一个新的 .class 文件

****
# 5. 方法引用


