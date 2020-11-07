# 定义

运行时创建一个实现了一组给定接口的新类

# 使用

要想创建一个代理对象，需要使用 Proxy 类的 newProxylnstance 方法。这个方法有三个参数

- 类加载器，现在直接使用 null 就行

- 一个 Class 对象数组，每个元素都是需要实现的接口

```
Class[] interfaces = new Class[] { Comparable.class }
```

- 一个调用处理器

```
class TraceHandler implements InvocationHandler
{
    private Object target;

    public TraceHandler(Object t)
    {
        target = t;
    }

    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable
    {
        // print method names and parameters
        ...

        // invoke actual method
        return m.invoke(target, args);
    }
}

Object value = ...;
InvocationHandler handler = new TraceHandler(value);
```

最后构造代理对象

```
Object proxy = Proxy.newProxyInstance(null, interfaces , handler)
```

# 注意

- 对于特定的类加载器和预设的一组接口来说，只能有一个代理类。也就是说，如果使用同一个类加载器和接口数组调用两次 newProxylustance 方法的话，那么只能够得到同一个类的两个对象，可以利用 getProxyClass 方法获得这个类

```
Class proxyClass = Proxy.getProxyClass(null, interfaces);
```

- 所有的代理类都覆盖了 Object 类中的方法 equals、hashCode 和 toString。如同所有的代理方法一样，这些方法仅仅调用了调用处理器的 invoke

---

参考源码

```
package com.kevinbai.proxy;

import java.lang.reflect.*;
import java.util.*;

public class ProxyTest
{
    public static void main(String[] args)
    {
        Object[] elements = new Object[1000];

        // fill elements with proxies for the integers 1 ... 1000
        for (int i = 0; i < elements.length; i++)
        {
            Integer value = i + 1;
            InvocationHandler handler = new TraceHandler(value);
            Object proxy = Proxy.newProxyInstance(null, new Class[] { Comparable.class } , handler);
            elements[i] = proxy;
        }

        // construct a random integer
        Integer key = new Random().nextInt(elements.length) + 1;

        // search for the key
        int result = Arrays.binarySearch(elements, key);

        // print match if found
        if (result >= 0) System.out.println(elements[result]);
    }
}

class TraceHandler implements InvocationHandler
{
    private Object target;

    public TraceHandler(Object t)
    {
        target = t;
    }

    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable
    {
        // print implicit argument
        System.out.print(target);
        // print method name
        System.out.print("." + m.getName() + "(");
        // print explicit arguments
        if (args != null)
        {
            for (int i = 0; i < args.length; i++)
            {
                System.out.print(args[i]);
                if (i < args.length - 1) System.out.print(", ");
            }
        }
        System.out.println(")");

        // invoke actual method
        return m.invoke(target, args);
    }
}
```
