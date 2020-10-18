不能被覆盖的类

```
public final class Executive extends Manager {
    private String name;

    public final String getName() {
        return this.name;
    }
}
```

- final 类，使用 final 修饰
- final 方法，使用 final 修饰
- final 类所有方法自动转为 final 方法（不包括域），final 方法可以存在于普通类
