
## 1. String类

>String 类代表字符串，Java 程序中的所有字符串字面值（如 "abc" ）都作为此类的实例实现，字符串是常量，它们的值在创建之后不能更改

****
## 2. String的不可变性

>`String` 对象一旦被创建，其内容是不可更改的，就算修改了字符串的值，实际上也是创建了一个新的对象，原对象保持不变，而接收变量的那个引用指向新的对象

![](images/String%20字符串/file-20250422093106.png)

>String 类是被 final 修饰的，它不允许被继承，其次 `String` 类内部定义了一个用来存储字符串内容的字节数组（在 Java 9 之后为 `byte[]`，在此之前为 `char[]`），该数组被 `private` 和 `final` 修饰，`final` 关键字保证了数组引用在对象创建后不可更改，也就是说它始终指向同一个数组，无法重新赋值，`private` 关键字限制了对数组的直接访问，防止外部程序绕过方法直接修改内容，所以对字符串进行修改操作（如拼接、截取等）时，实际上都会创建一个新的 `String` 对象，而不是在原对象上进行更改

****
## 3. 字符串常量池

> `String` 是不可变的对象，如果 JVM 每次都创建一个新的 `String` 对象，那么相同的就会在内存中存在两份，浪费空间，当 JVM 将所有的字符串字面量放入字符串常量池中后，如果已经存在相同的就直接返回已有引用避免重复创建具有相同内容的对象

所以字符串常量池是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

>如果是字符串字面量编译期就直接放入常量池，或者使用 `intern()` 方法在运行期将堆中的字符串手动加入常量池，但是所有的相同字符串字面量只会在常量池中存一份

****
## 4. 创建字符串的两种方式

### 4.1 字面量方式创建

```java
String str1 = "hello";
```

>Java 提供了一个字符串常量池（String Constant Pool），当使用双引号创建字符串时 JVM 会先在字符串常量池中查找是否已经存在相同内容的字符串对象，如果存在就直接返回引用，不存在就创建一个新的对象放进去

```java
String str1 = "abc";
String str2 = "abc";
System.out.println(str1 == str2); // true，两个变量引用的是常量池中的同一个对象
```

****
### 4.2 new 方式创建

```java
String str3 = new String("hello");
```

>这种方式会在堆内存中创建一个新的字符串对象，即使常量池中已经有相同内容的字符串，`new` 也会强制创建一个新的实例，这个 `new` 出来的字符串不会放到常量池中，而是放在堆中，  
>如果常量池中没有，那就在常量池中创建一个，然后再到堆中创建一个，而 `new String("hello")` 返回的是堆中对象的引用，  
>这个新的对象的内容和常量池一致，但它是堆中的独立对象  
>所以不管常量池中有没有该字符串，`new String("xxx")` 都会造成堆和常量池中各有一个副本

```java
String str1 = "abc";
String str2 = new String("abc");
System.out.println(str1 == str2); // false，引用地址不同
```

****
## 5. intern()

>首先它会在字符串常量池中查找是否有内容相同的字符串，如果有就返回常量池中的那个字符串引用，如果没有就创建一个引用指向该字符串，并返回这个引用（都在常量池中），所以 `intern()` 之后，返回的一定是常量池中的字符串引用

```java
String a = new String("hello");
String b = "hello";

System.out.println(a == b);          // false
System.out.println(a.intern() == b); // true
```

```java
String c = new String("hello word"); // 在常量池和堆中都会创建 "hello word" 
String d = c.intern(); // 此时常量池中已经拥有了 "hello word" 
String e = "hello word";  
System.out.println(c == d); // false; c 指向的是堆中的，d 指向的是常量池中的 
System.out.println(d == e); // true
```

>需要注意的是 Java7 之后字符串常量池就从方法区中移到了堆内存中，`intern` 也从将字符串的副本复制进常量池（位于永久代）变成存入引用，所以当常量池中没有对应的相同的字符串时，是可以存在 `c == d` 输出为 true 的情况的，因为使用 `new String("abc")` 时，常量池中只保存 `"abc"` 字面量本身的引用，而堆中新建的是另一个字符串对象，两者引用不同

```java
String s1 = new StringBuilder("inter").append("nTest").toString();  
String s2 = s1.intern();  
System.out.println(s1 == s2); // true
```

>因为通过拼接的方式，所以字符串常量池中最开始只会存在 inter ，当使用 intern 后 s2 指向的是常量池中指向堆中 internTest 的引用，也就是此时 s2 的引用和 s1 的引用相同

****

## 6. 字符串的拼接

### 6.1 使用 + 号拼接

>在 Java 中，`+` 运算符拼接字符串，其实底层是通过 `StringBuilder` 实现的，所以在拼接时会在堆中新建一个 `StringBuilder` 对象

#### 6.1.1 全是字面量的拼接

```java
String str = "Hello" + "World";
```

>拼接结果 `"HelloWorld"` 直接存入字符串常量池中，因为完全在编译期间完成，JVM 运行时只加载常量

![](images/String%20字符串/file-20250423164215.png)

>给 s2 加上 final 后，s2 成为了常量，它不再是变量，所以 s3 的拼接是在编译阶段完成的（可看作 s3 = "a" + "b"），输出结果为 ture

****
#### 6.1.2 包含变量的拼接

>这类拼接发生在运行时，编译器不会做合并优化，直接生成一个新的 `String` 对象放在堆中

```java
String a = "Hello";
String b = "World";
String c = a + b;

编译器会将这段代码翻译成类似下面的形式：

String c = new StringBuilder()
              .append(a)
              .append(b)
              .toString();
```

>这是由 Java 编译器（javac）生成的逻辑，它使用 `StringBuilder` 来提升性能，避免频繁创建新 `String` 对象

当字符串使用 `final` 修饰时可以让编译器当作常量来处理:

```java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

****
#### 6.1.3 常量折叠

>在编译过程中, javac 编译器会进行一个叫做常量折叠的代码优化, 它会把常量表达式的值求出来作为常量嵌在最终生成的代码中

对于 `String str3 = "str" + "ing";` 编译器会优化成 `String str3 = "string";`, 但并不是所有的常量都会进行折叠, 只有编译器在程序编译期就可以确定值的常量才可以(**引用的值在程序编译期是无法确定的，编译器无法对其进行优化**):

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

****
## 7.  字符串的实现

>Java 中的字符串都是 `String` 类的对象，而 `String` 的本质是用一个数组（`char[]` 或 `byte[]`）来存储字符数据

### 7.1 字面量字符串怎么变成 String 的

>例如，编译器会把 `"hello"` 当作一个字符串常量加入到 `.class` 文件的常量池中， JVM 会检查字符串常量池中是否已经有 `"hello"`，字面量字符串是 String 的对象 ，如果有就直接复用那个对象引用，如果没有就在堆中创建对象，并将引用存入常量池
>JDK 8 之前使用 `char[]` 保存字符，JDK 9+ 使用 `byte[]`

****
### 7.2 String底层由char[]优化成byte[]

>Java 9 以前，String 是用  `char []` 实现的，之后改成了 byte 型数组实现，并增加了 coder 来表示编码，从 `char[]` 到 `byte[]`最主要的目的是节省字符串占用的内存空间

****
### 7.3 字符串的字节数

### 1. Java 内部的内存占用（UTF-16）

>Java 中 `String` 实际存储为 `char[]`，而每个 `char` 是 2 个字节（16 位），采用 UTF-16 编码

```java
String s = "Hi";
// H = 2 字节; i = 2 字节
```

****
### 2. 编码后的字节数（UTF-8）

如果将字符串编码（如用于网络传输、写入文件），常用的是 UTF-8，而不是 UTF-16

```java
String s = "你好";
byte[] utf8 = s.getBytes(StandardCharsets.UTF_8);
System.out.println(utf8.length);  // 6
```

- '你' 在 Java 内部是 2 字节, 经过 UTF-8 编码后变成 3 字节; '好' 同理

****
### 3. 编译后 class 文件中常量池的字节表示

Java 编译器将字符串常量放入 class 文件的常量池中

```java
String s = "A你B";
```

在 `.class` 文件中的 UTF-8 表示中：

- 'A' = 1 字节
- '你' = 3 字节
- 'B' = 1 字节

****
### 4. 查看不同编码下的字节数

```java
String str = "你好abc";

byte[] utf8Bytes = str.getBytes("UTF-8");
byte[] utf16Bytes = str.getBytes("UTF-16");
byte[] gbkBytes = str.getBytes("GBK");

System.out.println("字符数: " + str.length()); // 5
System.out.println("UTF-8 字节数: " + utf8Bytes.length); // 9
System.out.println("UTF-16 字节数: " + utf16Bytes.length); // 12, 含 2 字节 BOM
System.out.println("GBK 字节数: " + gbkBytes.length); // 7
```

| 编码方式   | 中文字符字节数 | 英文字符字节数 | 说明        |
| ------ | ------- | ------- | --------- |
| UTF-8  | 3       | 1       | 变长编码      |
| UTF-16 | 2       | 2       | Java 内部编码 |
| GBK    | 2       | 1       | 中文系统常见    |

****
## 8. String的构造器

### 8.1 从char[]构造

```java
char[] c = {'H', 'e', 'l', 'l', 'o', '!'};
String str = new String(c);
```

![](images/String%20字符串/file-20250422212358.png)

>检查传入的字符属于哪个编码的范围，然后决定使用哪种编码，决定好后就把 `char[]` 转换成 `byte[]` 作为原始字符数据的编码存储形式存储在 `String` 对象中

![](images/String%20字符串/file-20250422213548.png)

****

### 8.2 从byte[]构造

```java
byte[] bytes = {72, 101, 108, 108, 111}; // "Hello"
String str = new String(bytes);
```

![](images/String%20字符串/file-20250422214021.png)

>`String` 会根据传入的 `charset` 创建对应的解码器，把 `byte[]` 中存储的编码解码成相应的字符，

![](images/String%20字符串/file-20250422214940.png)

>为了节省内存，Java就会把`char[]` 转换成 `byte[]` 存储在 `String` 对象中，所以这其实是个`byte[]`->`char[]`->`byte[]`的过程

>因为大部分情况下传入的`byte[]`是原始数据，它是经过编码后的结果，如果直接使用只会得到他的二进制码，所以在底层需要将它转换成人能够识别的字符，知道每个字节代表的是什么，然后再编码成计算机能够读懂的二进制码

****

## 9. 字符串的乱码

>字符串乱码本质上是使用错误的编码方式去解读字节流

![](images/String%20字符串/file-20250423135639.png)

>对一个字符串使用GBK的编码方式编码，然后再使用UTF-8进行解码，就会导致出现乱码

![](images/String%20字符串/file-20250423140214.png)

>通过 `String` 构造器对字节流进行编码和解码时可以使用 `""` 手动输入想用的编码方式，也可以使用底层自带的默认编码方式，

![](images/String%20字符串/file-20250423140319.png)

![](images/String%20字符串/file-20250423140522.png)

****

## 10. 字符串的常用方法

### 10.1 char charAt(int index)

>获取字符串中指定位置（索引处）的字符

![](images/String%20字符串/file-20250423142827.png)

****
### 10.2 int length()

>用于获取字符串中字符的数量

![](images/String%20字符串/file-20250423142953.png)

****
### 10.3 boolean isEmpty()

>用于判断字符串是否为空（即长度为 0）

![](images/String%20字符串/file-20250423143156.png)

****
### 10.4 boolean equals(Object anObject)

>判断两个字符串内容是否相等

![](images/String%20字符串/file-20250423143452.png)

****
### 10.5 boolean equalsIgnoreCase(String anotherString)

>忽略大小写的情况下判断两个字符串的内容是否相等

![](images/String%20字符串/file-20250423143709.png)

****
### 10.6 boolean contains(CharSequence s)

>判断当前字符串中是否包含某个子字符串

![](images/String%20字符串/file-20250423143944.png)

****

### 10.7 boolean startsWith(String prefix)

>判断当前字符串是否以某个字符串开头

![](images/String%20字符串/file-20250423144434.png)

****
### 10.8 boolean endsWith(String suffix)

>用来判断当前字符串是否以指定的后缀结尾

![](images/String%20字符串/file-20250423144544.png)

****
### 10.9 int compareTo(String anotherString)

>两个字符串按照字典顺序比较大小，返回一个整数，表示两个字符串的字典顺序，如果两个字符串相等返回 0， 如果当前字符串大于 `anotherString` 返回正整数，如果当前字符串小于 `anotherString` 返回负整数

![](images/String%20字符串/file-20250423144924.png)

>该方法通过逐个比较两个字符串的每个字符，进行相减，因为大小写字母的Unicode值不一样，所以需要区分大小写

****
### 10.10 int compareToIgnoreCase(String str)

>两个字符串按照字典顺序比较大小，比较时忽略大小写

![](images/String%20字符串/file-20250423145206.png)

****
### 10.11 int indexOf(String str, int fromIndex)

>从当前字符串的 `fromIndex` 下标开始往右搜索，获取当前字符串中 `str` 字符串的第一次出现处的下标

![](images/String%20字符串/file-20250423145542.png)

****
### 10.12 int lastIndexOf(String str, int fromIndex)

>从当前字符串的 `fromIndex` 下标开始往左搜索，获取当前字符串中 `str` 字符串的最后一次出现处的下标

![](images/String%20字符串/file-20250423150002.png)

****
### 10.13 byte[] getBytes()

>将字符串转换成字节数组，其实就是对字符串进行编码，默认按照系统默认字符集

![](images/String%20字符串/file-20250423151052.png)

****
### 10.14 byte[] getBytes(String charsetName)

>将字符串按照指定字符集的方式进行编码

![](images/String%20字符串/file-20250423151222.png)

****
### 10.15 byte[] getBytes(Charset charset)

>将字符串按照指定编码转换为字节数组

![](images/String%20字符串/file-20250423151440.png)

****
### 10.16 char[] toCharArray()

>将字符串转换字符数组

![](images/String%20字符串/file-20250423151612.png)

****
### 10.17 String toLowerCase()

>转小写

****
### 10.18 String toUpperCase()

>转大写

![](images/String%20字符串/file-20250423151843.png)

****
### 10.19 String concat(String str)

>将当前字符串和另一个字符串连接起来，因为字符串是不能改变的，所以返回的是源字符串和 str 拼接后的新字符串

![](images/String%20字符串/file-20250423153426.png)

**与 + 对比**

>这两种拼接的方式很类似，都会创建一个新的字符串，并且都放在堆中（用 + 拼接两个引用对象），不会进入常量池，所以进行多次的拼接操作时会创建多个 `String` 对象，导致占用很多内存  
>但 + 号拼接更灵活，不会出现空指针异常的情况

![](images/String%20字符串/file-20250423154103.png)

![](images/String%20字符串/file-20250423154125.png)

****
### 10.20 String substring(int beginIndex, int endIndex)

>截取字符串的一部分子字符串，范围是从 `beginIndex`（包括）到 `endIndex`（不包括）

![](images/String%20字符串/file-20250423155318.png)

****
### 10.21 static String join

>将多个字符串以某个分隔符连接

![](images/String%20字符串/file-20250423155953.png)

****
### 10.22 static String valueOf

>将非字符串转换成字符串

![](images/String%20字符串/file-20250423160735.png)

****
## 11. StringBuilder

>`StringBuilder` 是 Java 中一个用于构建可变字符串的类，在原有对象上修改，效率更高，通常用于频繁拼接字符串的场景

### 11.1 底层结构

![](images/String%20字符串/file-20250423173616.png)

![](images/String%20字符串/file-20250423173716.png)

>`StringBuilder` 的父类中的核心字段为`char[]`、`value` 和 `int count`，分别用来存储字符内容和记录当前字符串的长度，因为没有使用 `final` 修饰，所以它的实例对象的属性值是可以改变的，也就是创建的字符串是可变的

![](images/String%20字符串/file-20250423173927.png)

![](images/String%20字符串/file-20250423173905.png)

>在创建 `StringBuilder` 对象时会传入一个 16，这个就是 `StringBuilder` 的默认容量，如果后续拼接字符串大于默认长度后就会进行扩容操作

![](images/String%20字符串/file-20250423175631.png)

>先创建了一个长度为32的字符串，已经大于默认的容量了，所以肯定会进行扩容操作，底层会先对原容量（16）进行 ×2 + 2 的操作，此时的数组容量就成了 34，大于了32  
>当minGrowth > prefGrowth 时，可能就不是进行 ×2 + 2 的操作了，主要是看那个 growth（传进去后的参数名是minGrowth）

![](images/String%20字符串/file-20250423180752.png)

![](images/String%20字符串/file-20250423181137.png)

****
### 11.2 实现字符串的拼接

>字符串（`String`）是不可变的，每次拼接都会创建新的字符串对象，效率低下，为了提高性能就提供了 `StringBuilder` 类来实现对一个对象进行拼接

**字符串拼接使用 `+` 运算符**

```java
String str = "a" + "b";
String str2 = "c";
String str3 = str + str2;
```

>这个操作会创建四个对象，两个是常量池中的 `"ab"、"c"` ，两个是堆中的 `String` 对象的 `abc` 和 `StringBuilder` ，使用 + 拼接变量时底层会把字符串转换成 `StringBuilder` 对象，然后调用它的 `append()` 方法实现字符串的拼接

```
常量池：
 ├── "ab"
 └── "c"

堆内存：
 ├── StringBuilder对象
 │    └── char[] {'a','b','c',...}
 └── String对象（str3）
      └── byte[] {'a','b','c'}
```

**效率提升**

```java
String str = "";
for (int i = 0; i < 100000; i++) {
    str += str1; // 等价于 str = str + str1;
}
```

>这种情况就会产生很多个 `String` 对象和 `StringBuilder` 对象，效率十分低下，所以可以显示的使用 `appedn()` 方法，期间只会创建一个 `String` 对象 和`StringBuilder` 对象

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    sb.append(str1);
}
String str = sb.toString();
```

****
# 12. StringBuffer

>`StringBuffer` 是 Java 提供的一个可变字符序列类，用于高效地修改字符串内容，与 `String` 不同的是，`StringBuffer` 修改内容时不会新建对象，而是在原有基础上操作，因此效率更高，并且 `StringBuffer` 的所有方法都添加了 `synchronized` 所以它是线程安全的

![](images/String%20字符串/file-20250521223157.png)

构造方法：

```java
// 1. 空构造，默认初始容量 16
StringBuffer sb1 = new StringBuffer();

// 2. 指定初始容量
StringBuffer sb2 = new StringBuffer(50);

// 3. 以字符串作为初始内容
StringBuffer sb3 = new StringBuffer("Hello");
```

## 12.1 常用方法

1、添加内容

```java
append(String str) // 追加内容
insert(int offset, ...)  // 指定位置插入内容
```

```java
StringBuffer sb = new StringBuffer("hello");
sb.append("世界"); // hello世界
sb.insert(5, "哈哈"); // hello世界哈哈
System.out.println(sb);
```

2、 删除内容

```java
delete(int start, int end) // 删除从 start 到 end-1 的字符
deleteCharAt(int index) // 删除指定位置字符
```

3、替换内容

```java
replace(int start, int end, String str) // 替换指定区间内容
```

4、反转字符串

```java
reverse()
```

5、设置长度 / 容量

```java
setLength(int newLength); // 设置字符串长度（截断或填充空字符）
ensureCapacity(int minCap); // 保证容量不小于 minCap
```

6、取值

```java
charAt(int index); // 获取指定下标字符
length(); // 返回字符长度
substring(int start, int end); // 截取子串
```

****
## 12.2 String、StringBuilder、StringBuffer 三者的区别

| 特性/对比项    | String       | StringBuilder | StringBuffer |
| --------- | ------------ | ------------- | ------------ |
| 是否可变      | 不可变          | 可变            | 可变           |
| 线程安全性     | 不安全          | 不安全           | 安全（同步）       |
| 性能效率      | 较低（频繁创建新对象）  | 高（单线程场景推荐）    | 中等（加锁有性能损耗）  |
| 初始容量      | 自动           | 默认 16         | 默认 16        |
| 适合场景      | 小量字符串处理，常量拼接 | 单线程大量字符串拼接    | 多线程环境中的字符串处理 |
| 是否存常量池    | 是            | 否（堆内存）        | 否（堆内存）       |
| 是否可继承     | 可            | 可             | 可            |
| 创建对象时是否共享 | 常量池共用        | 新对象           | 新对象          |





