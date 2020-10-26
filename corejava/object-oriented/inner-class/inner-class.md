# 定义内部类

- 定义在类内部，与域平级

- 访问外围类对象

直接写外围类对象域或方法名称

如 beep

```
class TalkingClock
{
    private int interval;
    private boolean beep;

    public class TimePrinter implements ActionListener
    {
        public void actionPerformed(ActionEvent event)
        {
            System.out.println("At the tone, the time is " + new Date());
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
}
```

也可以使用 OuterClass.this 引用

```
TalkingClock.this.beep
```

- 不能有静态域名（final static 是可以的）和静态方法

# 实例化内部类

- 如果在外围类中，像普通类一样就行

```
ActionListener listener = new TimePrinter();
```

也可以使用 this.new 实例化

```
ActionListener listener = this.new TimePrinter();
```

- 如果在其它类中

```
TalkingClock.TimePrinter timePrinter = clock.new TimePrinter();
```

可以使用 OuterClass.InnerClass 声明变量

# 内部类原理

编译后都会成为常规的类文件，类名为 OuterClassName$InnerClassName。比如，TalkingClock 会生成两个类文件：TalkingClock.class 和 TalkingClock$TimePrinter.class

TalkingClock$TimePrinter.class 会生成一个域 this$0 指向 TalkingClock 的实例（内部类的构造方法会接收 TalkingClock 实例并初始化 this$0）

```
// javap -private com.kevinbai.innerClass.TalkingClock\$TimePrinter
Compiled from "InnerClassTest.java"
public class com.kevinbai.innerClass.TalkingClock$TimePrinter implements java.awt.event.ActionListener {
  final com.kevinbai.innerClass.TalkingClock this$0;
  public com.kevinbai.innerClass.TalkingClock$TimePrinter(com.kevinbai.innerClass.TalkingClock);
  public void actionPerformed(java.awt.event.ActionEvent);
}
```

外围类 TalkingClock 会生成一个静态方法 access$0 方便内部类访问外围类对象的域

```
// javap -private com.kevinbai.innerClass.TalkingClock
Compiled from "InnerClassTest.java"
class com.kevinbai.innerClass.TalkingClock {
  private int interval;
  private boolean beep;
  public com.kevinbai.innerClass.TalkingClock(int, boolean);
  public void start();
  // 待确认，这里是我手动加入的
  static boolean access$0(TalkingClock);
}
```

内部类中的

```
if (beep)
```

会被替换成

```
if (TalkingClock.access$0(outer))
```

---

参考源码

```
package com.kevinbai.innerClass;

import java.awt.*;
import java.awt.event.*;
import java.util.*;
import javax.swing.*;
import javax.swing.Timer;

public class InnerClassTest
{
    public static void main(String[] args)
    {
        TalkingClock clock = new TalkingClock(1000, true);
        clock.start();

        JOptionPane.showMessageDialog(null, "Quit program?");
        System.exit(0);
    }
}

class TalkingClock
{
    private int interval;
    private boolean beep;

    public TalkingClock(int interval, boolean beep)
    {
        this.interval = interval;
        this.beep = beep;
    }

    public void start()
    {
        ActionListener listener = this.new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }

    public class TimePrinter implements ActionListener
    {
        public void actionPerformed(ActionEvent event)
        {
            System.out.println("At the tone, the time is " + new Date());
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
}
```
