
>所有用static修饰的,直接通过 类名. 访问,不使用到对象

>static属于类级(放在堆中),所有对象都拥有这个属性,并且属性值一致,在内存空间上只有一份,节省内存开销

>static在类加载时就完成初始化

>虽然静态变量是类级别的,但是任然可以使用对象名去访问,但实际运行时和对象无关,所以即使引用为null,也仍然可以访问static变量

```
private static String country = "China";
...
```

```
Person person = new Person();
person = null;
person.country;//China
```

>所以空指针异常只出现在空引用访问实例相关的信息

>静态方法中不能使用this关键字,所以无法直接访问实例变量和实例方法


