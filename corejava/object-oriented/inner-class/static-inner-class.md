# 定义

- 有些时候，使用内部类是为了把一个类隐藏在另外一个类的内部，并不需要内部类引用外围类对象。为此，可以把内部类声明为 static，以便取消产生的引用

比如参考源码中的 Pair，其它程序员也很可能用这个名字，这样就会产生冲突，将其定义为 ArrayAlg 的内部公有类后，可以通过 ArrayAlg.Pair 访问

- 可以有静态域和方法

---

参考源码

```
package com.kevinbai.staticInnerClass;

public class StaticInnerClassTest
{
    public static void main(String[] args)
    {
        double[] d = new double[20];
        for (int i = 0; i < d.length; i++)
            d[i] = 100 * Math.random();
        ArrayAlg.Pair p = ArrayAlg.minmax(d);
        System.out.println("min = " + p.getFirst());
        System.out.println("max = " + p.getSecond());
    }
}

class ArrayAlg
{
    public static class Pair
    {
        private double first;
        private double second;

        public Pair(double f, double s)
        {
            first = f;
            second = s;
        }

        public double getFirst()
        {
            return first;
        }

        public double getSecond()
        {
            return second;
        }
    }

    public static Pair minmax(double[] values)
    {
        double min = Double.POSITIVE_INFINITY;
        double max = Double.NEGATIVE_INFINITY;
        for (double v : values)
        {
            if (min > v) min = v;
            if (max < v) max = v;
        }
        return new Pair(min, max);
    }
}
```
