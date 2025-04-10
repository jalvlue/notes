总结: 这就是 Hello World Java 了, 简单来说就是文件, 类, 函数, 变量, 基本输出, 最基础的语法规格

[[1.1 Java essentials]]

- 所有的 java 代码必须在类里, 类型必须跟源文件名相同
- 对于需要执行的类 (主类, 程序入口), 需要 `public static void main(String[] args)`
```java
// HelloWorld.java
public class HelloWorld {
	public static void main(String[] args) {
		System.out.println("hello world");
	}
}

/*
1. All code in Java must be part of a class.
2. We delimit the beginning and end of segments of code
   using { and }.
3. All statements in Java must end in a semi-colon.
4. For code to run we need public static void main(String[] args)
*/
```

- Java 变量需要先声明才能使用
- Java 是静态类型的, 变量类型不可变, 编译器会进行严格的类型检测
```java
public class HelloNumbers {
	public static void main(String[] args) {
		int x = 0;
		while (x < 10) {
			System.out.println(x);
			x = x + 1;
		}

		x = "horse";
	}
}

/*
1. Before Java variables can be used, they must be declared.
2. Java variables must have a specific type.
3. Java variable types can never change.
4. Types are verified before the code even runs!!!
*/
```


- 函数也必须声明定义在类中 (因此 java 中的所有函数都是方法)
- 函数的参数和返回值都必须有类型, 并且 java 函数只有一个返回值
```java
public class LargerDemo {
	/** Returns the larger of x and y. */
	public static int larger(int x, int y) {
		if (x > y) {
			return x;
		}
		return y;
	}

	public static void main(String[] args) {
		System.out.println(larger(-5.5, 10));
	}
} 

/*
1. Functions must be declared as part of a class in Java.
   A function that is part of a class is called a "method".
   So in Java, all functions are methods.
2. To define a function in Java, we use "public static".
   We will see alternate ways of defining functions later.
3. All parameters of a function must have a declared type,
   and the return value of the function must have a declared type.
   Functions in Java return only one value!
*/
```

