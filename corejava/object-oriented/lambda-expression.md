# lambda 表达式语法

```
(String first, String second) -> first.length() - second.length()
```

当根据上下文可以推测出参数类型时，可以省掉类型

```
Comparator<String> comp = (first, second) -> first.length() - second.length()
```

单个参数时可以省掉括号

```
ActionListener listener = event -> {
            System.out.println("hello");
            Toolkit.getDefaultToolkit().beep();
        };
```

没有参数时括号需要保留

```
() -> 0
```

不需要指定返回类型，但是表达式的返回值必须保持一致

如果一个 lambda 表达式只在某些分支返回一个值，而在另外一些分支不返回值，这是不合法的。例如，(int x) -> { if (x >= 0) return 1; } 就不合法

# 函数式接口

## 概念

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个 lambda 表达式。这种接口称为函数式接口（functional interface）

```
Comparator<String> comp = (first, second) -> first.length() - second.length();
Arrays .sort (words, comp);
```

## 方法引用

如果有现成的方法想传递给其它方法，可以使用方法引用。比如

```
Timer t = new Timer(1000, event -> System.out.println(event));
```

可以改为

```
Timer t = new Timer(1000, System.out::println);
```

这里

```
System.out::println
// 等同于
event -> System.out.println(event)
```

方法的引用形式有 3 种

- object::instanceMethod
- Class::staticMethod
- Class::instanceMethod

在前 2 种情况中，方法引用等价于提供方法参数的 lambda 表达式。前面已经提到，System.out::println 等价于 x -> System.out.println(x)；对于第 3 种情况，第 1 个参数会成为方法的目标。例如，String::compareToIgnoreCase 等同于 (x, y) -> x.compareToIgnoreCase(y)

另外，构造器引用语法如下

```
ClassName::new
```

```
ArrayList<String> names = . . .;
Stream<Person> stream = names.stream().map(Person::new);
```

## 常用函数式接口

![常用函数式接口](http://mweb.kevinbai.com/images/16034124708143.jpg)

![基本类型的函数式接口](http://mweb.kevinbai.com/images/16034125175433.jpg)


# 变量作用域

lambda 表达式有 3 个部分

- 一个代码块
- 参数
- 自由变量的值，这是指非参数而且不在代码中定义的变量

```
public static void repeatMessage(String text, int delay) {
    ActionListener listener = event -> {
        System.out.println(text);
        Toolkit.getDefaultToolkit().beep();
    };
    new Timer(delay, listener).start();

    JOptionPane.showMessageDialog(null, "Quit program?");
    System.exit(0);
}
```

其中，text 就是自由变量

自由变量的值在初始化后就不能被修改

```
// text 在 lambda 表达式外面被修改
public static void repeatMessage(String text, int delay) {
    text += "";
    ActionListener listener = event -> {
        System.out.println(text);
        Toolkit.getDefaultToolkit().beep();
    };
    ...
}
// text 在 lambda 表达式里面被修改
public static void repeatMessage(String text, int delay) {
    ActionListener listener = event -> {
        text += "";
        System.out.println(text);
        Toolkit.getDefaultToolkit().beep();
    };
    ...
}
// 这些都是不允许的，会报错
```

另外，lambda 表达式的体与嵌套块有相同的作用域，在一个 lambda 表达式中使用 this 关键字时，是指创建这个 lambda 表达式的方法的 this 参数

# 再谈 Comparator

Comparator 接口包含很多方便的静态方法来创建比较器

- 根据对象名称排序

```
Arrays.sort(people, Comparator.comparing(Person::getName));
```

- 与 thenComparing 串用

```
Arrays.sort ( people , Comparator.comparing(Person::getlastName) .thenConiparing(Person::getFi rstName));
```

如果两个人的姓相同，就会使用第二个比较器

- 可以为 comparing 和 thenComparing 方法提取的键指定一个比较器

```
Arrays.sort(people, Comparator.comparing(Person::getName, (s, t) -> Integer.compare(s.length(), t.length())));
```

- comparing 和 thenComparing 方法都有变体形式， 可以避免 int、long 或 double 值的装箱

```
Arrays.sort(people, Comparator.comparingInt(p -> p.getName().length()));
```

这比上面的方法更简单些

- 如果键函数可以返回 null, 可能就要用到 nullsFirst 或者 nullsLast 适配器

```
Arrays.sort(people, comparing(Person::getMiddleName, nullsFirst(naturalOrder())));
```

nullsFirst 方法需要一个比较器，naturalOrder 方法可以为任何实现了 Comparable 的类建立一个比较器。静态 reverseOrder 方法会提供自然顺序的逆序
