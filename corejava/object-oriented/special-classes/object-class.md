所有类的超类

包含的方法

# equals

## 使用

判断两个引用是否指向同一个对象

## 扩展使用

1. 检测 this 与 otherObject 是否引用同一个对象（这条语句只是一个优化。实际上，这是一种经常采用的形式。因为计算这个等式要比一个一个地比较类中的域所付出的代价小得多）
2. 检测 otherObject 是否为 null，如果为 null，返回 false。这项检测是很必要的
3. 比较 this 与 otherObject 是否属于同一个类。如果 equals 的语义在每个子类中有所改变，就使用 getClass 检测；如果所有的子类都拥有统一的语义，就使用 instanceof 检测
4. 将 otherObject 转换为相应的类类型变量
5. 现在开始对所有需要比较的域进行比较了。使用 = 比较基本类型域，使用 equals比 较对象域。如果所有的域都匹配，就返回 true; 否则返回 false

```
public class Employee
{
   private String name;
   private double salary;
   private LocalDate hireDay;

   public boolean equals(Object otherObject)
   {
      // a quick test to see if the objects are identical
      if (this == otherObject) return true;

      // must return false if the explicit parameter is null
      if (otherObject == null) return false;

      // if the classes don't match, they can't be equal
      // 如果每个子类的 equals 语义一致，比如根据 name 判定相等 ，可以使用 if !(otherObject instanceof Employee) return false;
      if (getClass() != otherObject.getClass()) return false;

      // now we know otherObject is a non-null Employee
      Employee other = (Employee) otherObject;

      // test whether the fields have identical values
      // name 和 hireDay 可以用 Objects.equals 方法改写，避免比较的某个对象为 null
      return name.equals(other.name) && salary == other.salary && hireDay.equals(other.hireDay);
   }
}
```

# hashCode

## 使用

返回对象的散列码，值为对象的地址（整型）

## 继承使用

```
public class Employee {
	...
    
    // 尽可能合理组织域的散列码
    // 可以使用 Objects.hashCode 以避免对象为空的情况
    // 也可以直接使用 Objects.hash(name, salary, hireDay) 生成
    public int hashCode() {
        return 7 * name.hashCode()
                + 11 * (new Double(salary)).hashCode()
                + 13 * hireDay.hashCode();
    }
}
```

重定义 equals 后，应该跟着重定义 hashCode，以便用户能将对象插入散列表中

# toString

## 使用

返回对象的字符串表示，值为类名 + @ + 散列码

## 继承使用

```
public class Employee {
	...
 
   public String toString()
   {
      return getClass().getName() + "[name=" + name + ",salary=" + salary + ",hireDay=" + hireDay
            + "]";
   }
}
```

在字符串与对象相加或者 System.out.println 的时候会自动调用对象的 toString() 方法

在编写下一层的子类的，前面 3 个方法都可以使用 super.f 利用父类 f 的结果
