
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
