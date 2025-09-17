# 1. ThreadLocal

## 1.1 ThreadLocal 有什么用

>`ThreadLocal` 提供了一个线程本地变量，每个线程都有一个自己独立的、初始化的变量副本。线程可以访问和修改自己的副本，而不会与其他线程的副本产生冲突。可以把它理解为一个以线程为 key，以存入的值为 value 的映射（Map）。每个线程通过 `ThreadLocal` 对象只能访问到映射给自己线程的那个值。

而 `ThreadLocal` 的核心价值在于 “线程隔离”，它解决了变量在线程间共享的问题，而是为每个线程创建了一份独立的实例。当创建一个 `ThreadLocal` 变量时，每个访问该变量的线程都会拥有一个独立的副本。线程可以通过 `get()` 方法获取自己线程的本地副本，或通过 `set()` 方法修改该副本的值，从而避免了线程安全问题。

主要用途：

1、存储线程上下文信息

这是 `ThreadLocal` 最常用的，在一个线程执行的上下文中，多个方法或组件可能需要访问同一个信息，但如果将这个信息作为参数在方法间层层传递，会导致代码冗余且难以维护。

```java
public class UserContextHolder {
    // 创建一个ThreadLocal变量，用于存储User对象
    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();
    
    public static void set(User user) {
    currentUser.set(user);
	}
	
	public static User get() {
	    return currentUser.get();
	}
    
    // 使用完后必须清除，防止内存泄漏
    public static void remove() {
        currentUser.remove();
    }
}

// 在 Filter 中
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    try {
	    // 获取用户信息
        User user = (User) request.getSession().getAttribute("user"); 
        UserContextHolder.set(user); // 存入ThreadLocal
        chain.doFilter(request, response);
    } finally {
        UserContextHolder.remove(); // 确保请求处理后一定清除
    }
}

// 在 Service 中，可以直接获取，无需方法参数传递
@Service
public class SomeService {
    public void someBusinessMethod() {
        User user = UserContextHolder.get(); // 直接获取当前用户
        System.out.println("Current user is: " + user.getName());
        // ... 业务逻辑
    }
}
```

2、避免在方法间传递通用参数

对于一些需要贯穿多个方法层级的通用参数，使用 `ThreadLocal` 可以极大地简化代码，避免出现 `methodA(String traceId, Param p)` -> `methodB(String traceId, Param p)` 这样的 “噪音” 参数。

3、提供线程安全的工具对象

有些类本身不是线程安全的（如 `SimpleDateFormat`），但如果每个线程都创建一个新的实例，开销会很大。使用 `ThreadLocal` 可以让每个线程只创建并共享一个自己的实例，既保证了线程安全，又避免了重复创建的开销。

```java
public class DateUtils {
    private static final ThreadLocal<SimpleDateFormat> DATE_FORMATTER =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
    
    public static String format(Date date) {
        return DATE_FORMATTER.get().format(date);
    }
}
// 每个线程调用 format 时，使用的都是自己线程的 SimpleDateFormat 实例，安全、高效。
```

****
## 1.2 ThreadLocal 原理

```java
public class Thread implements Runnable {
    // ...
    // 与此线程有关的 ThreadLocal 值，由 ThreadLocal 类维护 
    ThreadLocal.ThreadLocalMap threadLocals = null;
    // 与此线程有关的 InheritableThreadLocal 值，由 InheritableThreadLocal 类维护 
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    // ...
}
```

`Thread` 类内部持有一个名为 `threadLocals` 的 `ThreadLocalMap` 类型的成员变量，而 `ThreadLocalMap` 是 `ThreadLocal` 的一个**静态内部类**，它才是一个真正的键值对存储结构。它的Key 是 `ThreadLocal` 对象本身（弱引用），Value 是你存储的值。

| 类型                         | 强度  | 是否可被 GC 回收   | 典型用途                                   |
| -------------------------- | --- | ------------ | -------------------------------------- |
| **强引用（Strong Reference）**  | 最强  | 不可回收         | 普通对象引用，例如 `Object obj = new Object();` |
| **软引用（Soft Reference）**    | 较强  | 内存不足时回收      | 缓存系统，保证尽可能不被回收                         |
| **弱引用（Weak Reference）**    | 较弱  | 一旦 GC，就会回收   | ThreadLocalMap 的 key、弱引用队列、对象登记表       |
| **虚引用（Phantom Reference）** | 最弱  | 用于在对象被回收前做清理 | 垃圾回收前通知，ReferenceQueue 使用              |
所以，整个关系是这样的：

- 每个 `Thread` 都有一个自己独有的 `ThreadLocalMap`。
- 这个 `Map` 的 Key 是各个 `ThreadLocal` 变量。
- 这个 `Map` 的 Value 就是通过 `threadLocal.set(value)` 存储的值。

```java
public void set(T value) {
    // 1. 获取当前正在执行的线程
    Thread t = Thread.currentThread();
    // 2. 获取这个线程自己的 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    // 3. 如果 Map 不为空，直接将当前 ThreadLocal 对象作为 Key，要存的值作为 Value 放进去
    if (map != null) {
        map.set(this, value);
    } else {
        // 4. 如果 Map 还没初始化，则创建一个新的 ThreadLocalMap 并赋值给线程的  threadLocals 变量
        createMap(t, value);
    }
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals; // 直接返回线程的成员变量
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

```java
public T get() {
    // 1. 获取当前线程
    Thread t = Thread.currentThread();
    // 2. 获取线程的 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    // 3. 如果 Map 已存在
    if (map != null) {
        // 以当前 ThreadLocal 为 Key，尝试获取对应的键值对（Entry）
        ThreadLocalMap.Entry e = map.getEntry(this);
        // 4. 如果找到了，返回 Value
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 5. 如果 Map 不存在或者没找到值，则调用初始化方法并返回初始值，默认为 null
    return setInitialValue();
}
```

通过上面这些内容，足以得出结论：最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是 `ThreadLocalMap` 的封装，传递了变量值。 `ThrealLocal` 类中可以通过 `Thread.currentThread()` 获取到当前线程对象后，直接通过`getMap(Thread t)` 访问到该线程的 `ThreadLocalMap` 对象。每个 `Thread` 中都具备一个`ThreadLocalMap`，而` ThreadLocalMap` 可以存储以 `ThreadLocal` 为 key ，Object 对象为 value 的键值对。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //......
}
```

比如在同一个线程中声明了两个 `ThreadLocal` 对象的话， `Thread` 内部都是使用仅有的那个`ThreadLocalMap` 存放数据的，`ThreadLocalMap` 的 key 就是 `ThreadLocal` 对象，value 就是 `ThreadLocal` 对象调用 `set` 方法设置的值。

![](images/Java并发常见面试题总结（下）/IMG_9958.jpg)

****
## 1.3 ThreadLocal 内存泄露

什么是内存泄漏？

>内存泄露（Memory Leak）指的是：程序中已经动态分配的堆内存，由于某种原因未能被释放或无法被释放，造成系统内存的浪费。这会导致程序运行速度减慢，甚至最终导致  `OutOfMemoryError`。

ThreadLocal 内存泄露的根源在于 `ThreadLocalMap` 中 `Entry` 的特殊引用关系和线程的生命周期（尤其是线程池场景）。

![](images/Java并发常见面试题总结（下）/file-20250912175523.png)

1、`Entry` 的 Key 是弱引用。

将 Key 设计为弱引用是 JDK 开发者做的一种保护性措施，目的是当`ThreadLocal`对象失去外部所有强引用（除了 `ThreadLocalMap.Entry.key` 之外，程序代码里是否还保存着这个 `ThreadLocal` 对象的引用）时，在进行垃圾回收时，这个 `ThreadLocal` 对象本身可以被回收，避免 `ThreadLocal` 对象本身的内存泄露。

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k); // 调用 WeakReference 的构造方法，将Key（k）包装成一个弱引用
        value = v;
    }
}
```

如果不设计成弱引用的话，即使写了 `local = null;`，线程里的 `ThreadLocalMap` 仍然有一个 `Entry` 的 key 指向这个 `ThreadLocal` 对象，这样 GC 永远不会回收它，这就叫 `ThreadLocal` 对象本身泄露。

不过在 `ThreadLocalMap` 的设计中还有其它的保护措施，例如在 `ThreadLocal` 的 `get`、`set`、`remove` 方法中都会清除线程 `ThreadLocalMap` 里所有 `key` 为 `null` 的` value`。

例如 `set()` 方法，如果 `entry` 等于 `null`，则说明该索引位之前放的 key（`ThreadLocal` 对象） 被回收了，这通常是因为外部将 `ThreadLocal` 变量置为 `null`，又因为 `entry` 对 `ThreadLocal` 持有的是弱引用，一轮 GC 过后，对象被回收。在这种情况下，既然用户代码都已经将 `ThreadLocal` 置为 `null` 了，那么也就没打算再通过该对象作为 `key` 去取到之前放入 `ThreadLocalMap` 的 `value`，因此 `ThreadLocalMap` 中会直接替换掉这种不新鲜的 `entry`。

```java
private void set(ThreadLocal<?> key, Object value) {  
    Entry[] tab = this.table;  
    int len = tab.length;  
    int i = key.threadLocalHashCode & len - 1;  
  
    for(Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {  
        if (e.refersTo(key)) { // 找到相同 key，直接覆盖 value 
            e.value = value;  
            return;  
        }  
  
        if (e.refersTo((Object)null)) { // 发现 key 的弱引用已被清空
            this.replaceStaleEntry(key, value, i);  
            return;  
        }  
    }  
	// 没找到，直接插入新 Entry
    tab[i] = new Entry(key, value);  
    int sz = ++this.size;  
    if (!this.cleanSomeSlots(i, sz) && sz >= this.threshold) {  
        this.rehash();  
    }  
  
}
```

虽然有以上保护措施，但仍然存在内存泄露的风险，因为清理是机会性的，如果一个线程长期存活（例如线程池线程），并且 `ThreadLocal` 的 `key` 已经被丢掉，但该线程在之后很少做 `set/get` 操作触发清理，`entry` 的 `value` 仍有可能长时间保留，造成泄漏。

2、Entry 的 Value 是强引用

```text
当前线程 Thread
   │
   └── threadLocals (ThreadLocalMap)
         │
         └── Entry[] 数组
               │
               ├── Entry(key=WeakRef(ThreadLocal实例), value="abc")
               ├── Entry(key=WeakRef(ThreadLocal实例2), value="xyz")
               └── ...
```

当前线程 (Thread) -> 强引用 -> `threadLocals` (那个Map) -> 强引用 -> `Entry[]` -> 强引用 -> `Entry` 对象 -> 强引用 -> `Value`。

只要线程还活着（比如在线程池里等待），这条链子就是完整的，GC 在回收对象时，会从一些“根”（比如正在执行的方法中的局部变量）开始，检查哪些对象通过引用链是可达的，只要是可达的，就认为是“还有用”的，就不回收。现在，虽然 `Entry` 里的` Key` 是弱引用，并且指向的 `ThreadLocal` 对象被回收了，导致 `Key` 变成了 `null`，但这丝毫不影响上面那条牢牢拴着 `Value` 的强引用链，`Value` 依然被线程间接地管理，所以 GC 绝不会去回收它。

所以弱引用释放的对象是 `ThreadLocal`，因为 `Entry` 的 key 是一个弱引用，指向在代码里创建的 `ThreadLocal` 对象，GC 会回收的只是这个 `ThreadLocal` 对象本身，一旦 `ThreadLocal` 被回收，弱引用会变成 `null`。而那个 `Entry` 依然存在，因为它在 `Thread.threadLocals` 的 `Entry[]` 数组里被线程强引用着，并且 `Entry` 里的 `value` 仍然是强引用，暂时不会被 GC 回收。也就是说`ThreadLocal` 对象消失了，但 Entry 和 value 还在内存里。但弱引用并不是毫无意义的，当 `ThreadLocal` 被用户丢弃后，GC 可以把这个 key 干掉，这样 Map 在下一次被访问时，就能识别到这个 Entry 的 `key == null`，从而触发 `replaceStaleEntry` / `expungeStaleEntry`（调用 `set/get/remove`），清理掉 value 的强引用，弱引用不是直接解决 `value` 的泄漏，而是给了 Map 一个发现垃圾 key 的信号，让这个 `Entry` 等待延时清理。

3、发生内存泄露的情况

内存泄露发生的两个必要条件:

1. ThreadLocal 实例失去外部强引用
2. 存放 Value 的线程长时间存活且不再进行有效清理


| 场景                          | ThreadLocal引用状态 | 线程生命周期         | 是否会泄露？           | 解释                                                                                                                                           |
| :-------------------------- | :-------------- | :------------- | :--------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. 普通线程**                 | 失去强引用           | 执行完任务后**线程结束** | **不会**           | 线程死亡，`Thread`对象和内部的`ThreadLocalMap`会被整体回收，所有内存都被释放。                                                                                          |
| **2. 线程池 + 未调用remove**      | 失去强引用           | 线程**长期存活**于线程池 | **会**            | 同时满足了上述两个必要条件。                                                                                                                               |
| **3. 线程池 + 正确调用remove**     | 任何状态            | 线程长期存活         | **不会**           | `remove()`方法会手动断开对Value的强引用，并清理Entry，从根本上解决了问题。                                                                                              |
| **4. ThreadLocal是static的**  | **始终有**强引用（类引用） | 线程长期存活         | **不会泄露，但可能内存堆积** | 因为Key一直指向存活的`ThreadLocal`对象，Entry永远不会变废弃。但如果你不停创建新Value，旧的Value会被替换，可以被GC。但如果不再使用却未remove，Value会一直占用内存，这是一种**逻辑上的“内存堆积”**，而非严格意义的“泄露”，但同样有害。 |
| **5. ThreadLocal是非static的** | 随对象创建/销毁        | 线程长期存活         | **极易泄露**         | 当持有此 `ThreadLocal` 的类实例被回收后，`ThreadLocal`实例就失去了强引用，满足了条件一，极易进入泄露流程。                                                                          |
例如下面的代码，创建一个静态线程对象，全局只有一个实例，每个线程在执行 `threadLocal.set(TestClass) `时，会在当前线程的 `ThreadLocalMap` 里生成一个 `Entry`，`Entry.key` 指向 `threadLocal`，`Entry.value` 指向 `TestClass`（100M 的大对象）。虽然把 `TestClass` 对象置空了，但这里只是断开了局部变量的引用，`ThreadLocalMap` 的 `Entry.value` 仍然强引用着它，所以 GC 不会回收 `value`。如果不调用 `remove()`，`Entry` 会一直留在线程 Map 里，线程是线程池里的工作线程，`ThradLocal` 长时间存活，`Entry` 和 100M 的对象也长时间存活，导致内存泄露。

```java
public class ThreadLocalTestDemo {  
  
    private static ThreadLocal<TestClass> threadLocal = new ThreadLocal<>();  
  
  
    public static void main(String[] args) throws InterruptedException {  
  
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5, 5, 1, TimeUnit.MINUTES, new LinkedBlockingQueue<>());  
  
        for (int i = 0; i < 10; ++i) {  
            threadPoolExecutor.execute(new Runnable() {  
                @Override  
                public void run() {  
                    System.out.println("创建对象：");  
                    TestClass testClass = new TestClass();  
                    threadLocal.set(testClass);  
                    testClass = null; //将对象设置为 null，表示此对象不在使用了  
                    // threadLocal.remove();  
                }  
            });  
            Thread.sleep(1000);  
        }  
    }  
  
    static class TestClass {  
        // 100M  
        private byte[] bytes = new byte[100 * 1024 * 1024];  
    }  
}
```

****
## 1.4 如何跨线程传递 ThreadLocal 的值

`ThreadLocal` 的设计初衷就是实现线程隔离，每个 `Thread` 对象里有一个 `ThreadLocalMap`，存放这个线程自己的 `ThreadLocal` 变量和对应值，而父子线程是属于不同的 `Thread` 的，每个线程有自己的 `ThreadLocalMap`，互不共享。因此在异步场景下，父子线程的 `ThreadLocal` 值无法进行传递。

1、手动传递

在创建子线程之前，从父线程的 `ThreadLocal` 中取出值，作为参数传递给子线程的构造函数或 `Runnable`/`Callable` 任务。

```java
public class ManualPassing {  
    // 定义一个 ThreadLocal，用于存储用户 ID    
    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();  
  
    public static void main(String[] args) {  
        // 在父线程中设置值  
        threadLocal.set("User-123");  
  
        // 1. 从父线程(主线程)中取出值  
        String userId = threadLocal.get();  
  
        // 2. 将取出的值作为参数显式传递给子线程的任务  
        Runnable task = new MyTask(userId);  
        Thread childThread = new Thread(task);  
        childThread.start();  
    }  
  
    static class MyTask implements Runnable {  
        // 子线程任务持有父线程传递过来的数据  
        private final String userIdFromParent;  
  
        public MyTask(String userIdFromParent) {  
            this.userIdFromParent = userIdFromParent;  
        }  
  
        @Override  
        public void run() {  
            // 3. 在子线程中，如果需要，可以将其设置到子线程自己的 ThreadLocal 中  
            // 注意：这里是子线程自己的 ThreadLocal，与父线程的无关  
            threadLocal.set(userIdFromParent);  
            try {  
                System.out.println("在子线程中处理用户: " + threadLocal.get());  
                // ... 执行业务逻辑  
            } finally {  
                // 务必清理子线程的ThreadLocal，防止内存泄露  
                threadLocal.remove();  
            }  
        }  
    }  
}
```

2、使用 InheritableThreadLocal

`InheritableThreadLocal` 是 JDK 自带的一个类，它继承了 `ThreadLocal`。当父线程创建一个新的子线程时，子线程会自动继承父线程中 `InheritableThreadLocal` 变量所设置的值。

```java
public class InheritableThreadLocalDemo {  
    // 使用InheritableThreadLocal代替普通的ThreadLocal  
    private static final InheritableThreadLocal<String> threadLocal = new InheritableThreadLocal<>();  
  
    public static void main(String[] args) {  
        threadLocal.set("User-123");  
  
        // 父线程创建并启动子线程  
        Thread childThread = new Thread(() -> {  
            // 子线程可以直接获取到父线程设置的值  
            System.out.println("在子线程中获取用户: " + threadLocal.get()); // 输出：User-123  
            threadLocal.remove(); // 清理子线程的  
        });  
  
        childThread.start();  
    }  
}
```

在 `Thread` 的构造函数中，会检查父线程的 `inheritableThreadLocals`  变量是否存在。如果存在，它会创建一个新的 `ThreadLocalMap` 并将父线程 `inheritableThreadLocals` 中的所有 Entry 浅拷贝一份到子线程的 `inheritableThreadLocals` 中。

但它存在线程池失效问题，这是最大的问题。线程池的核心机制是线程复用，即线程创建后不会销毁，而是反复执行不同的任务。可 `InheritableThreadLocal` 只在线程创建时拷贝一次数据，后续提交给这个复用线程的任务，无法获取到真正提交它的那个父线程的数据，而是会拿到第一次创建线程时的旧数据，导致数据错乱。也就是说，只能在 `new Thread()` 时进行拷贝，这就存在很大的局限性了，并且， 既然是拷贝操作，那就存在对象的创建与销毁，这就存在一定的内存开销。

3、使用 TransmittableThreadLocal

`TransmittableThreadLocal` （简称 TTL） 是阿里巴巴开源的工具类，继承并加强了`InheritableThreadLocal`类，可以在线程池的场景下支持 `ThreadLocal` 值传递。官方文档：[https://github.com/alibaba/transmittable-thread-local](https://github.com/alibaba/transmittable-thread-local)

****
## 1.5 InheritableThreadLocal 原理


****

