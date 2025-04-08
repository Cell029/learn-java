
# 一.JVM规范中的运行时数据区

>1. The PC Register(程序计数器):记录正在执行的虚拟机字节码指令的地址
>2. Java Virtual Machine Stacks(Java虚拟机栈):存储栈帧.栈帧里面存储局部变量表,操作数栈,动态链接,方法出口信息等.
>3. Heap(堆):Java虚拟机管理的最大一块内存,存放Java对象实例和数组,堆是垃圾收集器收集垃圾的主要区域
>4. Method Area(方法区):存储已被虚拟机加载的类的信息,常量,静态变量等
>5. Run-Time Constant Pool(运行时常量池):方法区的一部分,存放编译期生成的字面量与符号引用(方法区中)
>6. Native Method Stacks(本地方法栈):在本地方法的执行过程中会使用到,与Java虚拟机栈十分相似,

**HotSpot**

>HotSpot是Oracle公司开发的,目前最常用的虚拟机实现,也是默认的Java虚拟机
# 二.jdk6的HotSpot

![](images/JVM虚拟机规范/file-20250408211046.png)