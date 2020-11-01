# 多态

一个对象变量可以指定多种实际类型的现在。在继承中就是指一个类型的变量可以指定该类型以及其子类的对象。

```
class Employee {
    
}

class Manager extends Employee {
    
}
```

可以这么使用

```
Employee e = new Employee();
Employee m = new Manager();
```

# 动态绑定

理解 d.f 方法的调用（其中 D 是 C 的子类，d 是 D 的实例）

## 编译阶段

1. 编译器根据可见性例举 D 以及 C 中的方法签名
2. 根据调用参数匹配要执行的方法（重载解析）
3. 如果 f 使用 private、static、final 方法修饰，那么编译器将可以准确地知道应该调用哪个方法，此为静态绑定；反之需要进行动态绑定，这里先生成调用某个签名的指令（比如 f(String) ）

## 运行阶段

当程序运行，并且采用动态绑定调用方法时，虚拟机一定调用与 d 所引用对象的实际类型最合适的那个类的方法。 如果 D 类定义了方法 f(String) 就直接调用它; 否则，将在 D 类的超类中寻找 f(String) 以此类推。

每次这样寻找效率低，虚拟机为每个类提前创建了方法表，列出了所有方法的签名和实际调用的方法，执行时虚拟机使用对应类的方法表去搜索就行，如下面的图

![](http://mweb.kevinbai.com/images/16027742303580.jpg)

![](http://mweb.kevinbai.com/images/16027742371393.jpg)

需要注意的是，域没有动态绑定的说法的

```
class A {
    int x = 1;
}

class B extends A {
    int x = 2;
}

public class Test {
    public static void main(String[] args) {
        A a = new B();
        System.out.println(a.x);
    }
}
```

这里输出的是 1，而不是 2，因为域只进行静态绑定。类似的还可以看个例子

```
class A 
{
    int x = 5;
} 
class B extends A 
{
    int x = 6;
} 
class SubCovariantTest extends CovariantTest 
{
    public B getObject() 
    {
       System.out.println("sub getobj");
       return new B();
    }
}

public class CovariantTest 
{
    public A getObject() 
    {
       System.out.println("ct getobj");
       return new A();
    } 
    public static void main(String[]args) 
    {
       CovariantTest c1 = new SubCovariantTest();
       System.out.println(c1.getObject().x);
    }
}
```

输出

```
sub getobj
5
```

理解了之前的例子，这个例子也好理解了

# 强制类型转换

将一个子类的引用赋给一个超类变量，编译器是允许的

```
Manager m = new Manager();
Employee e = m;
```

但将一个超类的引用赋给一个子类变量，必须进行类型转换

```
Employee e = new Manager();
Manager m = (Manager)e;
```
