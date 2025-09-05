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

