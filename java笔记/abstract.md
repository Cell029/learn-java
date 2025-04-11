
>abstract可以用来修饰类和方法,抽象方法必须存在抽象类中,继承抽象类就必须重写抽象类中的抽象方法,否则就要把子类也写成抽象类

# 一.抽象类

>抽象类和一般的类长得很像,只是加上了个修饰符,里面依然可以定义各种字段和方法,唯一有区别的就是方法不用再写上{}

```Java
public abstract class Animal {  
    private String name;  
    private int age;  
  
    public Animal() {  
    }  
    public Animal(String name, int age) {  
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
    public abstract void eat();  
}
```

>抽象类通常作为父类,常与多态相结合,所以必须提供父类该有的东西,确保能够让子类正常创建对象,但抽象类不能创建对象

>抽象类本质上并不是一个完整的类,该类里面包含的抽象方法只有声明却没有具体实现,Java语言是不允许一个对象具有无法调用的方法的,所以直接让抽象类无法new对象,确保程序运行安全,所以抽象类的作用就是为了被子类继承,而不是被实例化,它更像一个提供给子类使用的模板,具有一定的规则,但是没有具体的内容

# 二.抽象方法

>如果一个类中有特别的方法,它无法自己实现,需要由子类来确定具体的业务,那么这个方法在父类中就不需要有具体的实现,

