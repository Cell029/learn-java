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
    System.out.println(teacher.getName() + " " + teacher.getAge());  
  
    Teacher teacher1 = new Teacher("mike",20);  
    teacher1.dispaly();  
}
```


