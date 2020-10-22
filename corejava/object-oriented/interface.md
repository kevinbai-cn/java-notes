# 定义

```
public interface Named {
    String getName();
}
```

- 实例域，没有
- 实例方法，支持 default 方法
- 静态域，有，自动加上 final
- 静态方法，支持
- 作用域，域和方法默认 pulic，当类实现某个接口时，记得声名为 public，否则会有可见性变小的错误

# 使用

- 实现一个接口

```
class Person implements Named {
    public String getName() {
        return "P";
    }
}
```

- 实现多个接口

```
class Person implements Named, Named2 {

}
```

- 继承类的时候实现接口

```
class Student extends Person implements Named {

}
```

解决默认方法冲突

- 当实现多个接口时，如果存在相同的方法（其中一个是默认方法），则必须在类中进行覆盖
- 当继承某个超类和实现某个接口时，如果超类与接口相同的方法，已超类为准（类优先）

# 例子

- 接口与回调

关注 ActionListener 接口的 actionPerformed 方法

```
package timer;

/**
   @version 1.01 2015-05-12
   @author Cay Horstmann
*/

import java.awt.*;
import java.awt.event.*;
import java.util.*;
import javax.swing.*;
import javax.swing.Timer; 
// to resolve conflict with java.util.Timer

public class TimerTest
{  
   public static void main(String[] args)
   {  
      ActionListener listener = new TimePrinter();

      // construct a timer that calls the listener
      // once every 10 seconds
      Timer t = new Timer(10000, listener);
      t.start();

      JOptionPane.showMessageDialog(null, "Quit program?");
      System.exit(0);
   }
}

class TimePrinter implements ActionListener
{  
   public void actionPerformed(ActionEvent event)
   {  
      System.out.println("At the tone, the time is " + new Date());
      Toolkit.getDefaultToolkit().beep();
   }
}
```

- Compartor 接口

关注 compare 方法

```
class LengthComparator implements Comparator<String> {
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
}

public class Test {
    public static void main(String[] args) {
        String[] students = {"Peter", "Paul", "Mary"};
        Arrays.sort(students, new LengthComparator());
        System.out.println(Arrays.toString(students));
    }
}
```

- Cloneable 接口

关注 clone 方法

Cloneable 是个标记接口，标记接口一般不含方法，方便用来声明对象的类型

Object 的 clone 方法是 protected 作用域，所以实现接口时，会将这个方法覆盖为 public

区分浅拷贝和深拷贝

- 浅拷贝：拷贝原对象中的引用，某些引用可能指向可变对象
- 深拷贝：在浅拷贝基础上，对原对象中关于可变对象的引用进行深拷贝，这种情况需要实现 Clone 接口

```
// Employee.java
package clone;

import java.util.Date;
import java.util.GregorianCalendar;

public class Employee implements Cloneable
{
   private String name;
   private double salary;
   private Date hireDay;

   public Employee(String name, double salary)
   {
      this.name = name;
      this.salary = salary;
      hireDay = new Date();
   }

   public Employee clone() throws CloneNotSupportedException
   {
      // call Object.clone()
      Employee cloned = (Employee) super.clone();

      // clone mutable fields
      cloned.hireDay = (Date) hireDay.clone();

      return cloned;
   }

   /**
    * Set the hire day to a given date. 
    * @param year the year of the hire day
    * @param month the month of the hire day
    * @param day the day of the hire day
    */
   public void setHireDay(int year, int month, int day)
   {
      Date newHireDay = new GregorianCalendar(year, month - 1, day).getTime();
      
      // Example of instance field mutation
      hireDay.setTime(newHireDay.getTime());
   }

   public void raiseSalary(double byPercent)
   {
      double raise = salary * byPercent / 100;
      salary += raise;
   }

   public String toString()
   {
      return "Employee[name=" + name + ",salary=" + salary + ",hireDay=" + hireDay + "]";
   }
}
```

```
// CloneTest.java
package clone;

/**
 * This program demonstrates cloning.
 * @version 1.10 2002-07-01
 * @author Cay Horstmann
 */
public class CloneTest
{
   public static void main(String[] args)
   {
      try
      {
         Employee original = new Employee("John Q. Public", 50000);
         original.setHireDay(2000, 1, 1);
         Employee copy = original.clone();
         copy.raiseSalary(10);
         copy.setHireDay(2002, 12, 31);
         System.out.println("original=" + original);
         System.out.println("copy=" + copy);
      }
      catch (CloneNotSupportedException e)
      {
         e.printStackTrace();
      }
   }
}
```
