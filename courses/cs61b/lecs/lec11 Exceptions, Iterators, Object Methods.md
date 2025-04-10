[[6.1 Lists, Sets, ArraySet]]
[[6.2 Throwing Exceptions]]
[[6.3 Iteration]]
[[6.4 Equals]]

# exceptions
异常, 例如
- `NullPointerException`
- `IndexOutOfBoundsException`

- 抛出异常
```java
throw new RuntimeException("For no reason.");
```
- 捕获异常 `try/catch`, 在 `try` 代码块中的代码会先运行, 直到有异常抛出后, 停止, 转向 `catch` 代码块执行异常处理
- 如果 `try` 执行过程中没有异常发生, 那么将会直接跳过 `catch` 块
```java
try {
    dog.run()
} catch (Exception e) {
    System.out.println("Tried to run: " + e);
}
System.out.println("Hello World!");
```


# iterators and iterables
- 这两个接口的作用是让实现的类可以使用 `for (T i : collection)` 这样子的语法对集合进行简洁遍历
- 很接近很容易搞混的两个 java 接口
- `Iterable<T>` 是让一个类可迭代的接口, 必须返回一个 `Iterator<T>`
- `Iterator<T>` 是进行具体迭代操作的接口, 通常是某个需要迭代类的嵌套私有类, 由 `Iterator<T> iterator();` 方法新建
```java
public interface Iterator<T> {
  boolean hasNext();
  T next();
}

public interface Iterable<T> {
    Iterator<T> iterator();
}

for (int x : javaset) {
	System.out.println(x);
}

<==>

Iterator<Integer> seer = javaset.iterator();
while (seer.hasNext()) {
	System.out.println(seer.next())
}
```


# object methods
- java 中所有类的父类 `Object` 自己有几个方法
- `toString`, 返回一个实例对象的字符串表示
- `==` 比较符只是简单地对比两个参数是否有相同的值, 对于拷贝类型就是直接比较值, 对于引用类型就是比较内存地址
- `equals` 方法可以实现深度比较
- `o instanceof Dog uddaDog` 会检查 `Object o` 的动态类型是否为 `Dog`, 如果是则进行类型转换, 使用 `Dog uddaDog` 引用这个实例
- `of`


```java
public class Stone{
  public Stone(int weight){...}
}
Stone s = new Stone(100);
Stone r = new Stone(100);

@Override
public boolean equals(Object o){
  return this.weight == ((Stone) o).weight
}

@Override
public Boolean equals(Object o) {
	if (o instanceof Dog uddaDog) {
		return this.size == uddaDog.size;
	}
	return false;
}

public static <Glerp> ArraySet<Glerp> of(Glerp... stuff) {
	ArraySet<Glerp> returnSet = new ArraySet<Glerp>();
	for (Glerp x : stuff) {
		returnSet.add(x);
	}
	return returnSet;
}
ArraySet<String> asetOfStrings = ArraySet.of("hi", "I'm", "here");
```



