# 定义子类

```
class A {
    
}

class B extends A {
    
}
```

使用 extends 关键字

# 方法覆盖

```
class A {
    public void f() {
        System.out.println('A');
    }
}

class B extends A {
    public void f() {
        super.f();
        System.out.println('B');
    }
}
```

- B 中的 f 覆盖了 A 中的 f
- 覆不覆盖是根据方法签名来判定的，返回类型可以是父类方法返回类型或者它的子类（可协变返回类型）（如果不确定时可以使用 @Override 注解方法，当没覆盖成功时会编译报错）
- 子类方法的可见性要大于等于超类方法
- B.f 可以使用 super 调用超类的方法

# 子类构造方法

```
class A {
}

class B extends A {
    public B() {
        super();
    }
}
```

可以使用 super 调用父类的构造方法

# 继承层次

![](http://mweb.kevinbai.com/images/16027199169985.jpg)

- 由一个公共超类派生出来的所有类的集合被称为继承层次(inheritance hierarchy)（如上图）
- 在继承 层次中， 从某个特定的类到其祖先的路径被称为该类的继承链 ( inheritance chain ) 
