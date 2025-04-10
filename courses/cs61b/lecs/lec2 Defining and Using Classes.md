总结: java 是完全面向对象的, 对于一些不需要依赖类实例的东西, 可以使用静态修饰,

[[1.2 java object]]

对于 Java 中的类，最重要的就是
- 实例化
- 成员变量
- 方法
- Static vs non-static
- Manage complexity
	- Helper functions

# Overview
包含主方法的类称为主类，可以通过 `java` 命令执行
没有包含主方法的类则通过导出机制可以在其他类中使用，但是不能直接通过 `java` 命令执行

主类：
```java
public class DogLauncher {
	public static void main(String[] args) {
		Dog d = new Dog(15);

		Dog d2 = new Dog(100);

		// Dog bigger = Dog.maxDog(d, d2);
		Dog bigger = d.maxDog(d2);
		bigger.makeNoise();

		//System.out.println(d.binomen);
		//System.out.println(d2.binomen);
		//System.out.println(Dog.binomen);
//		d.makeNoise();
	}
} 
```

非主类：
```java
public class Dog {
	public int weightInPounds;
	public static String binomen = "Canis familiaris";

	/** One integer constructor for dogs. */
	public Dog(int w) {
		weightInPounds = w;
	}

	public void makeNoise() {
		if (weightInPounds < 10) {
			System.out.println("yip!");
		} else if (weightInPounds < 30) {
			System.out.println("bark.");
		} else {
			System.out.println("woooof!");
		}
	}

	public static Dog maxDog(Dog d1, Dog d2) {
		if (d1.weightInPounds > d2.weightInPounds) {
			return d1;
		}
		return d2;
	}	
} 
```

# Instantiate
- java 中类的实例化几乎都是通过 `new` 实现，例如 `Dog d = new Dog()`
- 构造函数
- 数组实例化，`Dog[] dogArr = new Dog[10]`

# Static/instance methods and variables
- static method 可以通过实例或者类进行调用，但是一般通过类调用更好，例如 `Math.sqrt`
- Instance method 只能通过实例进行调用
- 对于变量的操作同理
- `this` 关键字在类内可以指实例本身

# Command line arguments
- `java ArgsDemo these are command line arguments`
- `these are command line arguments` 会作为 `public static void main(String[] args)` 中，`args` 数组的成员
- 注意与 C 不同，函数名不被包括在 args 中

# Standard library
- 合理使用标准库
```java
public class ArgsSum {
	public static void main(String[] args) {
		int N = args.length;
		int i = 0;
		int sum = 0;
		while (i < N) {
			sum = sum + Integer.parseInt(args[i]);
			i = i + 1;
		}
		System.out.println(sum);		
	}
} 
```

