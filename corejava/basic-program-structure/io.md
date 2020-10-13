# 屏幕 IO

```
import java.util.Scanner;

public class SystemIo {

    public static void main(String[] args) {
        System.out.println("What is your name?");
        Scanner scanner = new Scanner(System.in);
        String name = scanner.next();

        System.out.println("How old are you?");
        int age = scanner.nextInt();

        System.out.println("Hello, " + name + ". Next year, you'll be " + (age + 1));
    }
}
```

可以使用 out.printf 格式化字符输出

# 文件 IO

```
import java.io.IOException;
import java.io.PrintWriter;
import java.nio.file.Paths;
import java.util.Scanner;

public class FileIo {

    public static void main(String[] args) throws IOException {
        Scanner scanner = new Scanner(Paths.get("/Users/kevinbai/Documents/Java/test/src/t.txt"), "UTF-8");
        PrintWriter out = new PrintWriter("/Users/kevinbai/Documents/Java/test/src/t2.txt", "UTF-8");
        while (scanner.hasNext()) {
            String s = scanner.next();
            System.out.println(s);
            out.println(s);
        }
        out.close();
    }
}
```
