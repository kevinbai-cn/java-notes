# 解释时

假定虚拟机要搜寻 com.horstmann.corejava.Employee 类文件

1. 在 jre/lib 归档文件中寻找类文件
2. 在 jre/lib/ext 归档文件中寻找类文件
3. 根据 CLASSPATH 寻找类文件

比如，如果 CLASSPATH 设置如下

```
export CLASSPATH=/home/user/classdir:.:/home/user/archives.jar
```

搜寻过程如下

1. 在 jre/lib 归档文件中寻找类文件
2. 在 jre/lib/ext 归档文件中寻找类文件
3. /home/user/classdir/com/horstmann/Employee.class
4. 当前目录下的 com/horstmann/Employee.class
5. /home/user/archives.jar 中的 com/horstmann/Employee.class

某个步骤成功即停止

PS：从 Java SE 6 开始， 可以在 JAR 文件目录中指定通配符， 类似

```
# 注意 * 使用引号括了起来，在 UNIX 中， 禁止使用 * 以防止 shell 命令进一步扩展
export CLASSPATH=/home/user/archives/'*'
```

# 编译时

假设源代码中使用了 Employee 类，而没有指出这个类所在的包，搜寻过程如下

1. 当前包中的 Employee
2. 检查所有的 import 语句，如果没找到，编译器报错；如果找到 1 个以上的类，如果不能确定，也要报错

```
// 假设 com.kevinbai.pack1 和 com.kevinbai.pack2 都有 Employee 类，现在要使用 Employee

// 报错
import com.kevinbai.pack1.*;
import com.kevinbai.pack2.*;

// 报错
import com.kevinbai.pack1.Employee;
import com.kevinbai.pack2.Employee;

// 使用 pack1 中的类
import com.kevinbai.pack1.Employee;
import com.kevinbai.pack2.*;
```

某个步骤成功即停止
