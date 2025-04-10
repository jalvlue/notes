[[4.3 Subtype Polymorphism]]

- 子类型多态: 由于有静态类型和动态类型的区分以及继承这种 `is-a` 关系, 因此可以使用静态类型变量指向多种动态类型实例, 调用方法时取决于动态类型的具体方法实现
- 例如: `deque` 调用 `deque.method` 时具体行为取决于子类的实现, 因此可以实现多种子类, 然后将静态类型为 `deque` 的变量指向多种子类, 从而调用多种 `deque.method`
- 利用 `interface` 实现函数多态的例子: 在 `Comparable` 这个接口中规划好了 `compareTo` 方法, 用于比较两个同类型的实例, 然后在 `max` 函数中, 使用 `Comparable` 类型作为入参以及出参, 利用了 `compare` 方法对两个实例进行比较
- 这里就很好利用了静态类型-对应多个动态类型的多态优势
- `Comparable<T>` 是 java 内置的让一个类型可比较的接口
- `Comparator<T>` 是另外一个标准库里的接口, 在类内可以定义多个私有嵌套类实现 `Comparator<T>` 从而可以通过这些私有类对该类进行多维度比较 (使用范围比 `Comparable<T>` 小)



```java
public interface Comparator<T> {
	public int compare(T x1, T x2);
}

public interface Comparable<T> {
    public int compareTo(T o);
}

public class Maximizer {
	public static Comparable max(Comparable[] items) {
		int maxDex = 0;
		for (int i = 0; i < items.length; i += 1) {
		    int cmp = items[i].compareTo(items[maxDex]);

			if (cmp > 0) {
				maxDex = i;
			}
		}
		return items[maxDex];
	}
}

public class Dog implements Comparable<Dog> {
    public String name;
    private int size;

    public Dog(String n, int s) {
        name = n;
        size = s;
    }

    @Override
    public int compareTo(Dog uddaDog) {
        //assume nobody is messing up and giving us
        //something that isn't a dog.
        return size - uddaDog.size;
    }

    public void bark() {
        System.out.println(name + " says: bark");
    }
}

public class DogLauncher {
    public static void main(String[] args) {
        Dog d1 = new Dog("Elyse", 3);
        Dog d2 = new Dog("Sture", 9);
        Dog d3 = new Dog("Benjamin", 15);
        Dog[] dogs = new Dog[]{d1, d2, d3};

        Dog d = (Dog) Maximizer.max(dogs);
        System.out.println(d.name);
    }
}
```