# 类名前面添加完整的包名

```
java.time.LocalDate today = java.time.LocalDate.now();
```

# import

```
// 导入特定类
import java.time.LocalDate;
// 导入整个包
import java.time.*
```

当导入的类名冲突时，

- 如果导入的是两个包造成，可以使用导入特定类的方式导入冲突类名

```
import java.sql.*;
import java.util.*;
import java.util.Date;
```

- 否则，只能在类名前面添加完整的类名

```
// 多余的类前添加完整包名
Date d1; // 使用 import 导入 java.util.Date
java.sql.Date d2;
// 或者，都添加完整包名
java.util.Date d1;
java.sql.Date d2;
```

可以使用 import 导入静态域或者方法

```
// 导入语句
// 导入特定静态域或者方法
import static java.lang.System.out;
// 导入所有静态域或者方法
import static java.lang.System.*;

// 使用
out.println(1);
```
