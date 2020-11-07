# 概念

所有的基本类型都对应了一个类，这些类称为类包装器。包括 Byte、Short、Integer、Long、Float、Double、Boolean、Character、Void

一定程度上是范型类的类型参数不支持基础类型，所以引入了这个

# 自动装箱

```
Integer i = 3;
```

自动编译成

```
Integer i = Integer.valueOf(3);
```

自动装箱规范要求
- byte、 介于 -128 ~ 127 之间的 short、int 和 long
- boolean
- char 127
被包装到固定的对象中

```
// 打印 false
Integer a = 1000;
Integer b = 1000;
System.out.println(a == b);
// 打印 true
Integer a = 100;
Integer b = 100;
System.out.println(a == b);
```

# 自动拆箱

```
int i = ic; // ic 是个 Interger 对象
```

自动编译成

```
int i = ic.intValue();
```

# 自动装箱或拆箱例子

```
ArrayList<Integer> list = new ArrayList<>();
// 自动装箱
list.add(1)
// 自动拆箱
int i = list.get(0)
// 自动拆箱后装箱
Integer n = 0;
n++;

// 如果在一个条件表达式中混合使用 Integer 和 Double 类型，Integer 值就会拆箱，提升为 double，再装箱为 Double
Integer n = 1;
Double x = 2.0;
System.out.println(true? n : x); // 打印 1.0
```

# 其它

对象包装器封装了些常用方法

```
int x = Integer.parseInt("1");
```
