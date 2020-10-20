# 概念

使用类似数组，在增加或者删除元素时有自动调节容量的功能

# 使用

```
// ArrayList 范型类，Employee 类型参数
ArrayList<Employee> employees = new ArrayList<>();
// 添加元素
employees.add(new Employee());
// 修改元素
employees.set(0, new Employee());
// 删除元素
employees.remove(0);
// 数组元素个数
System.out.println(employees.size());
```

# 类型化与原始数组的兼容性

原始数组列表可以不指定类型参数

```
ArrayList objs = new ArrayList();
```

将类型化的数组列表赋值给原始数组列表没有问题

```
objs = employees
```

反过来

```
employees = objs
```

会抛出警告：[unchecked] 未经检查的转换。编译时指定 -Xlint:unchecked 可以查看具体信息。即使使用 `(ArrayList<Employee>)` 进行强制类型转换也有警告
