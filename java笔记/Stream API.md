
# 1. 概述

>Java 流（Stream）是一连串的元素序列，元素一个一个地经过加工与处理，流式编程的概念基于函数式编程的思想，每个加工步骤就是一个函数式操作（如 `map`, `filter`, `sorted` 等），它不改变原集合而是产生一个新的结果，但可以简化代码，提高可读性和可维护性
>
>流并不会存储数据，它更像是一种封装好的工具，数据在流中被流动处理，每次使用流只能完成一次遍历，操作完成后立即失效，再次使用需要重新创建新的流

# 2. Stream 流的 3 种操作类型

- 创建Stream
- Stream中间处理
- 终止Steam

![](images/Stream%20API/file-20250514201131.png)

>每个 Stream 管道操作类型都包含几个 API 方法，通过这些 API 方法将数据处理好
## 2.1 开始管道

>主要负责新建一个 Stream 流，或者基于现有的数组、List、Set、Map 等集合类型的对象创建出新的 Stream 流

### 1. 从集合创建 Stream

>集合类都提供了 `stream()` 方法

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");  
// 创建顺序流  
Stream<String> stream = list.stream();  
System.out.println(stream); // java.util.stream.ReferencePipeline$Head@b4c966a
```

2、从数组创建 Stream

>使用 `Arrays.stream()` 或 `Stream.of()`

```java

```







