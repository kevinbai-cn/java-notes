# 简单

## 整型 byte short int long
## 浮点型 float double（小数默认为 double）
## 布尔值
## 字符

# 复杂

## 数组

- 初始化

```
// 数字会初始化为 0
// 字符会初始化为 '\u0000'
// 对象会初始化为 null
int[] a = new int[10];

int[] b = {1, 2, 3}
```

也可以使用匿名数组赋值

```
int[] a = new int[] {1, 2, 3};
```

- 常用方法

```
Arrays.toString(a)
Arrays.copyOf(a, a.length)
Arrays.sort(a);
```

- 多维数组初始化

```
int[][] aa = new int[10][10];
int[][] bb = {{1, 2, 3}, {4, 5, 6}};
```

第二维的数组的长度可以不同

```
// 手动指定第二维的数组
int[][] aa = new int[10][];

int[][] bb = {{1, 2, 3}, {4, 5, 6, 7}};
```

- 多维数组常用方法

```
Arrays.deepToString(bb)
```

## 枚举类型

```
enum Size { SMALL, MEDIUM, LARGE, EXTRA LARCE };
Size s = Size.MEDIUM
```

# 特殊

## null

# 标准库

## String

- 不可变

共享字符串常量

- 判断相等

使用 quals 判断相等

```
String greeting = "Hello";
if (greeting.equals("Hello")) System.out.println(1); // true
if (greeting.substring(0, 3).equals("Hel")) System.out.println(2); // true
if (greeting == "Hello") System.out.println(3); // true
if (greeting.substring(0, 3) == "Hel") System.out.println(4); // false
```

不输出 4 是因为只共享字符串常量，而 +、substring 等中间结果并不共享

- 区分码点和代码单元（1 个字符）

## BigInteger

实现任意精度的整数运算

```
BigInteger a = BigInteger.valueOf(100);
BigInteger b = BigInteger.valueOf(200);
BigInteger c = a.add(b);
BigInteger d = a.multiply(b);
```

# BigDecimal

实现任意精度的浮点数运算

使用和 BigInteger 类似
