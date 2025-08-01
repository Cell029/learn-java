
# 1. volatile 关键字

## 1.1 如何保证变量的可见性

>在多线程环境下，每个线程都有自己的[工作内存](多线程.md#11.2.1%20两个内存)（CPU 缓存），线程对变量的操作先在工作内存中进行，主内存中的变量值可能未被及时更新。所以一个线程对变量的修改，其他线程可能看不到，出现数据不同步的问题。

而 `volatile` 的作用就是保证可见性，当一个变量被 `volatile` 修饰后，线程对该变量的写操作会立刻刷新回主内存，并且线程对该变量的读操作，会直接从主内存中读取，而不是从工作内存缓存中读取，这样就保证了一个线程修改了变量，其他线程马上能看到修改后的最新值。

![](images/Java并发常见面试题总结（中）/b3022fe945455236b39bae092b0a06c2_720.png)

![](images/Java并发常见面试题总结（中）/29d9f0eb06dc71c89d140e323edca9eb.png)

Java 内存模型（JMM）对 `volatile` 有如下要求：

- 禁止指令重排序优化：对 `volatile` 变量的读写操作不会被重排序，以此保证操作顺序。
- 内存屏障：编译器和 CPU 会在 `volatile` 变量的读写前后插入内存屏障，确保：
    - 写 `volatile` 变量之前的操作必须先执行，写操作会立刻刷新到主内存。
    - 读 `volatile` 变量之后的操作必须后执行，读操作必须从主内存加载最新值。

```java
private volatile boolean flag = false;

Thread A:
    flag = true; // 写操作，立刻刷新到主内存

Thread B:
    while (!flag) {
        // 如果 flag 不是 volatile，可能一直读缓存，死循环
        // 如果 flag 是 volatile，能及时看到 flag 变为 true，跳出循环
    }
```

但需要注意的是：`volatile` 只能保证可见性，不保证原子性。例如：

```java
volatile int count = 0;
count++;
// 以下整组操作不是原子的，分别为读-改-写三步
int temp = count; // 读取 volatile count（从主内存读取）  
temp = temp + 1; // 自增
count = temp; // 写回 volatile count（写回主内存）
```

所以即使 `count` 由 `volatile` 修饰，但多个线程同时执行 `count++` 依然会导致数据竞争。若既需要保证数据的可见性，又需要操作的原子性，就可以使用 `synchronized`，进入 `synchronized` 的线程，一定能看到其他线程对同一锁保护下共享变量的修改，因为当一个线程进入 `synchronized` 的[临界区](多线程.md#12.%20synchronized)时，会清空本地工作内存（线程栈中的变量副本），并从主内存中重新读取共享变量的值。当线程退出同步块（释放锁）时，会把对共享变量的修改刷新回主内存，确保别的线程能看到这些变化。

****
## 1.2 volatile 如何禁止指令重排

JVM 和 CPU 为了优化程序执行效率，会对指令进行重排序，因为一个汇编指令也会涉及到很多步骤，每个步骤可能会用到不同的寄存器，也就是说，CPU 有多个功能单元（如获取、解码、运算和结果），一条指令也分为多个单元，那么第一条指令执行还没完毕，就可以执行第二条指令，前提是这两条指令功能单元相同或类似，所以一般可以通过指令重排使得具有相似功能单元的指令接连执行来减少流水线中断的情况。例如：

```java
int a = 1;
int b = 1;
a = a + 1;
b = b +1 ;
```

```java
int a = 1;
a = a + 1;
int b = 1;
b = b +1 ;
```

前者的性能可能就优于后者，因为如果局部变量连续声明使用，有可能在编译器优化时更容易保持寄存器命中，减少从栈读取。

当声明一个 `volatile` 变量时，JVM 会在编译后插入内存屏障指令来约束指令的重排序。在 Java 中，`Unsafe` 类提供了三个内存屏障相关的方法：

```java
// 屏蔽之后的所有读操作被重排到该屏障之前，等价于读取屏障（LoadLoad）
public native void loadFence();
// 屏蔽之前的所有写操作被重排到该屏障之后，等价于写入屏障（StoreStore）
public native void storeFence();
// 屏蔽所有读写操作的重排，等价于读写全屏障（LoadLoad + StoreStore + StoreLoad）
public native void fullFence();
```

- `loadFence()`：读不能穿透
- `storeFence()`：写不能穿透
- `fullFence()`：读写都不能穿透

`volatile` 与内存屏障的关系：

```java
// 写 volatile 变量
store to memory
storeStore barrier
write to volatile variable
storeLoad barrier

// 读 volatile 变量
read from volatile variable
loadLoad barrier
load from memory
```

具体内存指令：

- 在每个 `volatile` 写操作的前面插入一个 StoreStore 屏障，确保前面的普通写操作（非 volatile）在内存中对其他线程可见之后，才执行 `volatile` 变量的写

```java
sharedData = 123; // 普通变量
flag = true; // volatile 变量
```

如果没有 StoreStore 屏障，CPU 可能会重排序为：

```java
flag = true;
sharedData = 123;
```

- 在每个 `volatile` 写操作的后面插入一个 StoreLoad 屏障，防止 `volatile` 写与后续的任何读/写操作发生重排序

```java
// 确保写入后，所有线程读取到 object 时，它的字段已经初始化完成
object = new MyObject(); // volatile 写
```

- 在每个 `volatile` 读操作的后面插入一个 LoadLoad 屏障，防止在 `volatile` 读之后的普通读操作被重排序到 volatile 读之前

```java
boolean f = flag; // volatile 读
// 防止 CPU 把 x = sharedData 提前到 f = flag 之前执行
int x = sharedData; // 普通读
```

- 在每个 `volatile` 读操作的后面插入一个 LoadStore 屏障，防止 `volatile` 读操作之后的普通写被重排序到 volatile 读之前

```java
boolean f = flag; // volatile 读
// 防止 sharedData = 123 提前到 flag 被读取之前执行
sharedData = 123; // 普通写
```

也就是说，`volatile` 变量的读写操作底层实际通过插入内存屏障来实现可见性，从而控制 JVM 与 CPU 的缓存一致性。

****
## 1.3 volatile 可以保证原子性吗

>`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的。

```java
public class VolatileAtomicityDemo {
    public volatile static int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        VolatileAtomicityDemo volatileAtomicityDemo = new VolatileAtomicityDemo();
        for (int i = 0; i < 5; i++) {
            threadPool.execute(() -> {
                for (int j = 0; j < 500; j++) {
                    volatileAtomicityDemo.increase();
                }
            });
        }
        // 等待 1.5 秒，保证上面程序执行完成
        Thread.sleep(1500);
        System.out.println(inc);
        threadPool.shutdown();
    }
}
```

正常情况应该是五个线程都对 `inc` 累加 500 次，所以最终输出应该为 2500，但实际输出不足 2500，可见 `volatile` 无法保证原子性。实际上，`inc++` 其实是一个复合操作，包括三步：

1. 读取 inc 的值
2. 对 inc 加 1
3. 将 inc 的值写回内存

`volatile` 是无法保证这三个操作是具有原子性的，有可能导致下面这种情况出现：

1. 线程 1 对 `inc` 进行读取操作之后，还未对其进行修改。线程 2 又读取了 `inc`的值并对其进行修改（+1），再将`inc` 的值写回内存
2. 线程 2 操作完毕后，线程 1 对 `inc`的值进行修改（+1），再将`inc` 的值写回内存

这也就导致两个线程分别对 `inc` 进行了一次自增操作后，`inc` 实际上只增加了 1。其实，如果想要保证上面的代码运行正确也非常简单，利用 `synchronized`、`Lock` 就可以了。

****

