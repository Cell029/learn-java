# 1. ArrayList 和 Array（数组）的区别

`ArrayList` 内部基于动态数组实现，比 `Array`（静态数组） 使用起来更灵活：

- `ArrayList`会根据实际存储的元素动态地扩容或缩容；而 `Array` 被创建后长度不可变
- `ArrayList` 可以使用泛型来确保类型安全，`Array` 则不可以
- `ArrayList` 中只能存储对象，对于基本类型数据，需要使用其对应的包装类（如 Integer、Double 等）；`Array` 可以直接存储基本类型数据，也可以存储对象
- `ArrayList` 支持插入、删除、遍历等常见操作（但底层是数组，效率仍然较低），并且提供了丰富的 API 操作方法，如 `add()`、`remove()`等；`Array` 只是一个固定长度的数组，只能按照下标访问其中的元素，不具备动态添加、删除元素的能力
- `ArrayList`创建时不需要指定大小；而`Array`创建时必须指定大小

****
# 2. ArrayList 和 Vector 的区别

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]`存储，适用于频繁的查找工作，但线程不安全 
- `Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，线程安全但效率较低，已不推荐使用

****
# 3. Vector 和 Stack 的区别

- `Vector` 和 `Stack` 两者都是线程安全的，都是使用 `synchronized` 关键字进行同步处理
- `Stack` 继承自 `Vector`，在其基础上增加了栈的操作方法（如 `push()`、`pop()`），遵循先进后出（LIFO）的规则

****