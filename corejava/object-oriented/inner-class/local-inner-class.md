# 定义

- 定义在方法中的类

- 除了可以访问外部类，还可以访问局部变量（外部类方法中的局部变量）（和 labmda 表达式类似，不能对外变量进行修改）

# 使用

在作用域内使用 new 关键词即可

# 原理

看下 TalkingClock$1TimePrinter.class

```
// javap -private com.kevinbai.localInnerClass.TalkingClock\$1TimePrinter
Compiled from "LocalInnerClassTest.java"
class com.kevinbai.localInnerClass.TalkingClock$1TimePrinter implements java.awt.event.ActionListener {
  final boolean val$beep;
  final com.kevinbai.localInnerClass.TalkingClock this$0;
  // 这里我从 IntelliJ IDEA 看是
  // TalkingClock$1TimePrinter(TalkingClock var1, boolean var2)
  // javap 这里应该有问题   com.kevinbai.localInnerClass.TalkingClock$1TimePrinter();
  public void actionPerformed(java.awt.event.ActionEvent);
}
```

对使用到的局部变量（外部类方法中的局部变量）会作为内部类对象的 final  域，这里是 val$beep（内部类的构造方法会接收 beep 参数并初始化 val$beep）

---

参考源码

```
package com.kevinbai.localInnerClass;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Date;

public class LocalInnerClassTest {
    public static void main(String[] args)
    {
        TalkingClock clock = new TalkingClock();
        clock.start(1000, true);

        JOptionPane.showMessageDialog(null, "Quit program?");
        System.exit(0);
    }
}

class TalkingClock
{
    public void start(int interval, boolean beep)
    {
        class TimePrinter implements ActionListener
        {
            public void actionPerformed(ActionEvent event)
            {
                System.out.println("At the tone, the time is " + new Date());
                if (beep) Toolkit.getDefaultToolkit().beep();
            }
        }

        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }
}
```
