
>abstract可以用来修饰类和方法,抽象方法必须存在抽象类中,继承抽象类就必须重写抽象类中的抽象方法,否则就要把子类也写成抽象类

# 一.抽象类

>抽象类和一般的类长得很像,只是加上了个修饰符,里面依然可以定义各种字段和方法,唯一有区别的就是方法不用再写上`{}`

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

>抽象类通常作为父类,常与多态相结合,所以必须提供父类该有的东西,确保能够让子类正常创建对象,所以抽象类仍然拥有构造方法,但抽象类不能创建对象

>抽象类本质上并不是一个完整的类,该类里面包含的抽象方法只有声明却没有具体实现,Java语言是不允许一个对象具有无法调用的方法的,所以直接让抽象类无法new对象,确保程序运行安全,所以抽象类的作用就是为了被子类继承,而不是被实例化,它更像一个提供给子类使用的模板,具有一定的规则,但是没有具体的内容

>抽象类可以没有抽象方法,主要是用来强制继承和防止实例化

>抽象类中也可以定义普通实例方法,因此抽象类可以被看作半抽象的

# 二.抽象方法

>如果一个类中有某个方法，它的具体实现需要依赖子类的业务逻辑，也就是说父类无法自己确定这个方法该怎么做,那么这个方法就不应该写出方法体（即没有 `{}`},并把这个方法声明成抽象方法,只写方法的声明,由子类负责实现

>Java语言规定抽象方法只能是一个声明,它没有具体的实现,而写上了`{}`就证明它是一种实现,不管有没有写内容在`{}`中


```Java
public abstract void eat();
```

**abstract与private\final\static的冲突**

>如果一个方法用abstract修饰了,那么就不能再用private\final\static修饰,反之亦然

>1. private表示私有的,它本省就是不能被继承的,与abstract想要的效果冲突,所以必定会产生冲突  
>2. final表示最终的,当final修饰方法时,此方法就不能被重写,而抽象方法的本质就是强制子类重写方法,所以也必然冲突  
>3. static修饰的东西是属于类级别的,它本身就不允许被重写,所以也必然冲突

**抽象方法并不是延迟执行,而是延迟实现**

>延迟执行指的是代码已经写好了,但是我现在不执行,等被调用的时候才执行,而抽象方法是属于根本不能执行,因为它根本没写具体的实现  
>延迟实现指的是父类有个方法,这个方法不是写好了不执行,而是直接不写,等子类补上了才能执行

```Java
abstract class Demo {
    abstract void sayHello();

    void test() {
        sayHello();//如果直接调用的话这里会因为没实现方法而直接崩溃
    }
}
```

>所以抽象方法是怎么做都没说清,而不是晚点再来做

# 三.继承抽象类

```Java
public class Cat extends Animal{  
    public Cat() {  
    }  
    public Cat(String name, int age) {  
        super(name, age);  
    }  
  
    @Override  
    public void eat() {  
        System.out.println(this.getName() + "小猫爱吃猫条");  
    }  
}
```

>Cat类继承抽象Animal类,必须实现全部抽象方法的重写,使用子类继承抽象类可以让代码只依赖于抽象类,而不是具体的某个类,当新增子类时不需要修改原有的父类代码,只需要实现代码的重写即可,完全不会影响到父类,并且调用时所有子类的方法名都是一样的,用起来很方便

>这种继承关系常用来处理实例对象不同但行为类似的逻辑