- Primitives: 原子类型, 值类型, 变量在内存中的位置就是存放变量值对应的位模式
	- 在 java 中是 `byte, short, int...`
	- `int x = 1` 定义了一个值类型变量
	- `int y = x` 会做拷贝(拷贝值, 也就是位模式)
- References: 引用类型
	- `Walrus w = new(Walrus)`, 
	- 所有非原子类型的变量都是引用类型的
	- 引用类型都会有一个 64 位的变量, 它的值就是实际存储实例内存地址, 也就是引用实际存储
	- `Walrus new = w`, 此时只是复制了引用的 64 位变量, 也就是两个引用指向了一个实例
- 参数传递: 函数参数是原子类型时, 直接值传递, 在函数内拷贝一个副本, 传递引用类型则会影响到实际实例
- IntList: 整形的裸递归单链表
```java
public class IntList {
	public int first;
	public IntList rest;

	public IntList(int f, IntList r) {
		first = f;
		rest = r;
	}

	/** Return the size of the list using... recursion! */
	public int size() {
		if (rest == null) {
			return 1;
		}
		return 1 + this.rest.size();
	}

	/** Return the size of the list using no recursion! */
	public int iterativeSize() {
		IntList p = this;
		int totalSize = 0;
		while (p != null) {
			totalSize += 1;
			p = p.rest;
		}
		return totalSize;
	}

	/** Returns the ith item of this IntList. */
	public int get(int i) {
		if (i == 0) {
			return first;
		}
		return rest.get(i - 1);
	}

	public static void main(String[] args) {
		IntList L = new IntList(15, null);
		L = new IntList(10, L);
		L = new IntList(5, L);

		System.out.println(L.get(100));
	}
} 
```