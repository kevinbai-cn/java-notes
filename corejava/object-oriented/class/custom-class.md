# 实例域、实例方法

```
public class Employee {
    private int id = 10;

    public int getId() {
        return this.id;
    }
}
```

与实例绑定

# 静态域、静态方法

```
class Math {
    public static final double PI = 3.14159265358979323846;

    public static double pow(double a, double b) {
        return StrictMath.pow(a, b);
    }
}
```

与类绑定

使用类名调用

# 对象构造

## 实例域初始化过程

1. 默认域初始化

数字初始化为 0，布尔值初始化为 false，对象引用初始化为 null

2. 显式域初始化
3. 初始化块
4. 构造方法
	- 默认有个空构造方法，如果自己定义了一个以上的构造器，需要考虑添加自己添加空参数的构造方法
	- 可以使用 this() 调用另一个构造方法

**第 2、3 步的顺序取决于其定义在类中的顺序**

## 静态域初始化过程

与实例域初始化过程类似，不过第 3 步是静态初始化块，且只执行到这个位置

1. 默认域初始化
2. 显式域初始化
3. 静态初始化块

只在第一次加载类时才执行

* see also 关于对象构造的例子可参考 4.6.7 一节中的代码 *

# 对象析构

Java 有 GC 机制，不需要析构方法。

finalize 方法在对象被清理前调用，不过一般不使用于回收其它资源，因为不确定这个方法什么时候被调用

# 方法

## 传值

不管是基本数据类型还是对象引用，都是按值调用。对于后者，由于引用指的是同一个对象，所以可能导致原来的对象发生改变

* see also 详细描述可参考 4.5 *

## 重载

编译器根据方法签名选择要调用的方法，方法签名由方法名、参数类型、参数个数组成

# 作用域

- public，所有类可访问
- protect，子类或者当前包中的类可访问
- private，当前类可访问
- 默认，只能被当前包中的类访问

需要注意的是，一个方法可以访问所属类的所有对象的私有域和方法

```
class Employee {
    private int id = 10;

    public int getId() {
        return this.id;
    }

    public boolean equals(Employee other) {
        return this.id == other.id;
    }
}

public class Test {

    public static void main(String[] args) {
        Employee employee1 = new Employee();
        Employee employee2 = new Employee();
        System.out.println(employee1.equals(employee2));
    }
}
```
