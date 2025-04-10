>super代表的是当前对象中的父类型的特征,当子类继承了父类时,super可以看作当前对象的一部分,所以super不能使用在静态上下文中

>当子类声明了一个和父类相同的字段时,父类的字段依然存在,并且会被分配在子类的对象的内存中,所以子类的对象中包含了两个同名的字段,分别独立存在,通过不同的编译类型来访问对应的字段,所以想要在子类中访问同名的父类的字段,可以选择父类引用指向子类或者使用super.字段名来访问

```Java
public class Person {  
    private String name;  
    private int age;  
  
    public Person() {  
  
    }  
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
}

public class Teacher extends Person{  
    private String name;  
    private int age;  
  
    public Teacher() {  
  
    }  
    public Teacher(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    public void dispaly() {  
        System.out.println(this.name);  
        System.out.println(this.age);  
  
        System.out.println(super.getName());  
        System.out.println(super.getAge());  
    }  
}
```

```Java
public static void main(String[] args) {  
    Teacher teacher = new Teacher();  
    teacher.setName("jack");  
    teacher.setAge(22);  
    System.out.println(teacher.getName() + " " + teacher.getAge());//jack 22  
  
    Teacher teacher1 = new Teacher("mike",20);  
    teacher1.dispaly();//mike 20 null 0  
}
```

![](images/super关键字/file-20250410200029.png)

>查看上面程序的输出,子类和类有相同的字段,当创建子类对象是,会给子类和父类的字段都赋值,只不过一个是默认值,一个是初始值,但是他们都存在子类对象的内存中,如果子类没有重写get和set方法,那么在子类对象赋值时,赋的就是父类的字段,如果子类没有写有参构造器的话,赋的也是父类的字段

>需要注意的是,每个子类对象都拥有自己独立的一份父类字段,通过super查看到的父类字段不过是它的默认值(创建时调用的无参构造器),这些父类字段是复制出来的,并不会和其他子类对象共享

```Java
public class Teacher extends Person{  
    private String name;  
    private int age;  
  
    public Teacher() {  
  
    }  

    public void dispaly() {  
        System.out.println(this.name);  
        System.out.println(this.age);  
  
        System.out.println(super.getName());  
        System.out.println(super.getAge());  
    }  
}

Teacher teacher2 = new Teacher();  
teacher2.setName("jhon");  
teacher2.setAge(21);  
teacher2.dispaly();//null 0 jhon 21
```

![](images/super关键字/file-20250410201206.png)

>当子类里面不存在set,get和有参构造器时,想要给字段赋值就只能利用父类的set和get方法,此时给赋值的字段就变成了父类的字段,所以输出的当前对象的字段是null和0(成员变量默认值)

>所以,当子类和父类中定义了相同名称的字段,就要使用this和super来进行区分,否则根据引用类型或者使用的set\get方法属于谁来选择使用的字段

**子类的构造器中默认隐含一个super()**

>子类继承父类后能够使用父类的前提是父类的结构完整,也就是需要在父类的字段完成初始化后才能把父类的字段给到子类的对象,但是这个过程并不会创建一个父类的对象而是直接作为子类对象的一部分存进内存中,所以在子类创建对象时会先激活父类,让父类先加载一下(必须在第一行)

>如果子类没有显示的写super(...)/this(...)时才会执行隐含的super()

**父类方法也是在调用super()时放进子类对象中的吗?**

>不是被放在对象中,而是放在了虚方法表中(元空间),不过父类的方法是在类加载时就准备好的,此时已经创建好了父类的虚方法表,并不是在调用super()时才进行加载方法,super()只是用来确保父类的字段能够被正确初始化而已

>因为子类继承了父类,所以当子类被加载时父类也会被加载,与有没有创建子类对象无关

>所以类的字段的初始化是在对象被创建时才开始的,并不是在类被加载时开始的,因为字段属于实例对象,不属于类,而方法是属于类的,所以是在类加载时就初始化的,和静态变量一样

**this可以输出但是super不可以输出**

>this是当前对象的引用,它指向当前的实例对象在堆中的地址,它是一个明确存在的引用变量,作为变量它当然可以输出  
>super不是一个真实的对象也不是一个实际的引用变量,它更像一种语法,没有实际的内存,只是作为一种媒介来访问父类中被隐藏的字段或方法,所以它并不能被输出,并且一个子类可能是第n层子类,它的上面有很多比他大的类,可以通过super去访问直接父类,但不能跨级访问,所以super没有明确的指向性,也就没法输出

**有参构造器中的super(参数)**

>前面有说过,构造器中默认隐含一个super(),用来调用父类的无参构造器初始化父类,所以在有参构造器中手动写一个super(),就是为了给父类的字段手动赋值,如果父类没有无参构造器,那这个就必须写进子类的有参构造器了

>如果父类没有无参构造器,那么子类也不能写无参构造器,所以只要new对象,Object的无参构造器一定会执行

```Java
public class Teacher extends Person {  
    private double salary;  
  
    public Teacher(String name, int age, double salary) {  
        super(name, age);  
        this.salary = salary;  
    }  
}
```

>那一些重写方法中的 super.方法 是干嘛的呢?这个就是可以在父类方法的基础上进行扩展,避免改变重写整个方法