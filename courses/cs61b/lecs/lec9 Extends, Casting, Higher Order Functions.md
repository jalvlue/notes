[[4.2 Extends, Casting and Higher order funtions]]

- 继承和实现类似, 都可以从上级获取一些已有的代码
- 实现可以看作是一种约定, 并可以用来进行多态
- 继承则会获得父类所有共有的方法
- 父类的构造函数
	- 构造函数不会被继承, 但是子类可以通过 `super()` 调用父类的构造函数
	- 父类的默认构造函数 (无参构造函数) 会被子类隐式调用, 如果需要调用有参的父类构造函数, 则需要显式地使用 `super(x)` 来调用

```java
/** SList with additional operation printLostItems() which prints all items
  * that have ever been deleted. */
public class VengefulSLList<Item> extends SLList<Item> {
    SLList<Item> deletedItems;

    public VengefulSLList() {
        super();
        deletedItems = new SLList<Item>();
    }

    public VengefulSLList(Item x) {
        deletedItems = new SLList<Item>();
    }

    @Override
    public Item removeLast() {
        Item x = super.removeLast();
        deletedItems.addLast(x);
        return x;
    }

    /** Prints deleted items. */
    public void printLostItems() {
        deletedItems.print();
    }

    public static void main(String[] args) {

		VengefulSLList<Integer> vs1 = new VengefulSLList<Integer>(0);
		vs1.addLast(1);
		vs1.addLast(5);
		vs1.addLast(10);
		vs1.addLast(13);
		// vs1 is now: [1, 5, 10, 13] 


		vs1.removeLast();
		vs1.removeLast();
		// After deletion, vs1 is: [1, 5]

		// Should print out the numbers of the fallen, namely 10 and 13.
		System.out.print("The fallen are: ");
		vs1.printLostItems();
	}
} 
```

- 继承是 `is-a` 的关系, 子类对象就是一种父类对象
- java 中的 `Object` 类是很特殊的一个类, 所有的类都继承了 `Object`
- Java 支持强制类型转换, 
	- 一般情况下, 可以将一个子类对象向上转换为父类, 也就是让这个对象的动态类型是子类, 但是静态类型是父类 `Poodle largerPoodle = (Poodle) maxDog(frank, frankJr);`
	- 有时候也可以非常谨慎地向下转换
- 
- 使用继承抽象来控制程序的复杂度

- Java 中也有高阶函数, 但是实现的方法比较复杂, 需要通过接口来实现, 同时, 不是直接传递方法, 而是将方法写进某个类中, 然后将这个类的实例传递进去, 然后再调用这个实例的该方法

```java
/**
 * Created by hug on 2/6/2017.
 * Demonstrates higher order functions in Java.
 */
public class HoFDemo {
    public static int do_twice(IntUnaryFunction f, int x) {
        return f.apply(f.apply(x));
    }

    public static void main(String[] args) {
        IntUnaryFunction tenX = new TenX();
        System.out.println(do_twice(tenX, 2));
    }
}


/**
 * Created by hug on 2/6/2017.
 * Represent a function that takes in an integer, and returns an integer.
 */
public interface IntUnaryFunction {
    int apply(int x);
}


/**
 * Created by hug on 2/6/2017.
 */
public class TenX implements IntUnaryFunction {
    /** Returns ten times the argument. */
    public int apply(int x) {
        return 10 * x;
    }
}
```

