# Course theme
- 两个目的
	- 上手系统编程, 工具, 方法...
	- 导论, 了解计算机基本思想, 拓展到其他细分领域

# Great Reality #1
- `ints` are not integers, `floats` are not reals
- 计算机做的只是非常接近的模拟, 不能达到实际的数学定义
![[Pasted image 20241211211852.png]]
- `int32` 溢出了!
![[Pasted image 20241211212257.png]]


# Great Reality #2
- `you've got to know assambly`
- 汇编语言是直接命令 cpu 操作的语言
- 需要看懂 c 编译器的中间汇编代码, 了解程序实际执行过程
- `intel x86-64 assambly`

# Great Reality #3
- `memory matters`
- 性能, 内存管理

```c
#include <stdio.h>

typedef struct {
        int a[2];
        double d;
} struct_t;

double fun(int i) {
        volatile struct_t t;
        t.d = 3.14;
        t.a[i] = 1073741824;
        return t.d;
}

int main(void) {
        printf("%f\n", fun(0));
        printf("%f\n", fun(1));
        printf("%f\n", fun(2));
        printf("%f\n", fun(3));
        printf("%f\n", fun(4));
        printf("%f\n", fun(5));
        printf("%f\n", fun(6));
}
```
![[Pasted image 20241211213817.png]]
- 栈溢出!
![[Pasted image 20241211214152.png]]
- `c/cpp` 并不是内存安全的语言, 如果内存访问越界产生了段错误 `segmentation fault`, 操作系统将直接杀死整个进程, 而不会打印出完整的报错代码运行堆栈信息

# Great Reality #4
- `there's more to performance than asymptotic complexity, constant factors matter too!`
- 底层常数项优化也很重要

![[Pasted image 20241211214734.png]]
- 著名的封面


# Great Reality #5
- `computers do more than execute computations`
- 网络 io, 分布式系统
![[Pasted image 20241211215415.png]]


# labs

- `data representation`, bits operations, arithmetic, assembly language programs
	- `L1 data lab`, manipulating bits
	- `L2 boom lab`, defusing a binary bomb
	- `L3 attack lab`, the basics of code injection attacks
- `the memory hierarchy`, memory technology, memory hierarchy, caches, disks, locality
	- `L4 cache lab`, cache simulator and optimizing for locality
- `exceptional control flow`, hardware ex exceptions, processes, process control, unix signals, nonlocal jumps
	- `L5 shell lab`, your own unix shell
- `virtual memory`, virtual memory, address translation, dynamic storage allocation
	- `L6 malloc lab`, your own malloc package
- `networking, concurrency`, unix system level IO, network programming, web servers, concurrency, threads, IO multiplexing
	- `L7 proxy lab`, your own web server


# Welcome and Enjoy!
