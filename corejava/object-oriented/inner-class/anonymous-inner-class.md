# 定义、使用

对于局部内部类，如果只需要实例化一次，可以不指定其类名，这种类叫匿名内部类

通常语法格式

```
// 继承于父类
new SuperType(constructtion params) {
  inner class data and methods
}
// 实现某个接口
new InterfaceType() {
  inner class data and methods
}
```

比如

```
public void start(int interval, boolean beep)
{
    ActionListener listener = new ActionListener() {
        public void actionPerformed(ActionEvent e) {
            System.out.println("At the tone, the time is " + new Date());
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    };
    Timer t = new Timer(interval, listener);
    t.start();
}
```

这里实现了 ActionListener 接口

# 注意

- 匿名内部类没有构造方法，只能将构造参数传递给父类的构造方法

- 静态方法获取当前类名

```
new Object(){}.getClass().getEnclosingClass()
```

- 之前 Java 程序员习惯的做法是用匿名内部类实现事件监听器和其他回调，现在最好还是使用 lambda 表达式

---

参考源码

```
package com.kevinbai.anonymousInnerClass;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Date;

public class anonymousInnerClassTest {
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
        ActionListener listener = new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                System.out.println("At the tone, the time is " + new Date());
                if (beep) Toolkit.getDefaultToolkit().beep();
            }
        };
        System.out.println(new Object(){}.getClass().getEnclosingClass());
        Timer t = new Timer(interval, listener);
        t.start();
    }
}
```
