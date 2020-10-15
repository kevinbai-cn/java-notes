# 在文件头使用 package 语句

```
package com.kevinbai.pack1;

public class Employee {
    private int id = 10;

    public int getId() {
        return this.id;
    }
}
```

包路径与文件路径相对应

# 包作用域

- public，可以被当前包或者其它包中的类使用
- 默认只能被当前包中的类使用
