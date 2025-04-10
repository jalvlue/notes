
- 使用接口实现多态
```java
public interface List<Item> { ... }

default public void print() {...}

public AList<Item> implements List<item> {...}
```

- 静态类型 vs 动态类型
Java 中的每个变量都有一个静态类型。这是在声明变量时指定的类型，并在编译时进行检查。每个变量也有一个动态类型;这个类型在变量被实例化时被指定，并在运行时被检查。
```java
Thing a;
a = new Fox();
```

动态方法选择规则是，如果我们有一个静态类型 `X` 和一个动态类型 `Y` ，那么如果 `Y` 重写了来自 `X` 的方法，那么在运行时，我们使用 `Y` 中的方法。
