[[RISC-V Calling Conventions]]
TODO: 实践课, 去看视频
# C 程序到汇编程序的转换
- 计算机只能理解二进制编码的汇编语言代码
- C 程序需要先编译成二进制文件也就是 `.obj, .o` 文件
- 汇编非常底层, 程序没有组织架构可言, 没有流程控制, 汇编函数只是一个个的标签


# RISC-V vs x86
- `riscv` 是精简指令集, 指令少而且简单, 提供了更高的灵活性
- `x86` 是复杂指令集 `cisc`, 指令多, 历史包袱重, 很多指令都做了不止一件事情


# gdb 和汇编代码执行
```asm
.section .text
.global sum_to
/*
int sum_to(int n) {
	int acc = 0;
	for (int i = 0; i <= n; i++) {
		acc += i;
	}
	return acc;
}
*/

sum_to:
	mv to, a0			# t0 <- a0
	li a0, 0			# a0 <- 0
  loop:
    add a0, a0, t0		# a0 <- a0 + t0
    addi t0, t0, -1		# t0 <- t0 - 1
    bnez t0, loop		# if t0 != 0: pc <- loop
    ret
```
- 一个简单的 `c->asm` 函数代码

```ad-faq
学生提问：这里面.secion，.global，.text分别是什么意思？

TA：global表示你可以在其他文件中调用这个函数。text表明这里的是代码，如果你还记得XV6中的图3.4，每个进程的page table中有一个区域是text，汇编代码中的text表明这部分是代码，并且位于page table的text区域中。text中保存的就是代码。

学生提问：.asm文件和.s文件有什么区别？

TA：我并不是百分百确定。这两类文件都是汇编代码，.asm文件中包含大量额外的标注，而.s文件中没有。所以通常来说当你编译你的C代码，你得到的是.s文件。如果你好奇我们是如何得到.asm文件，makefile里面包含了具体的步骤。
```
![[image 2 1.avif]]
- gdb 有非常多的快捷指令, 需要的时候去搜就好


# RISC-V 寄存器
- ![[Pasted image 20240724093758.png]]

```ad-note
寄存器是CPU或者处理器上，预先定义的可以用来存储数据的位置。寄存器之所以重要是因为汇编代码并不是在内存上执行，而是在寄存器上执行，也就是说，当我们在做add，sub时，我们是对寄存器进行操作。所以你们通常看到的汇编代码中的模式是，我们通过load将数据存放在寄存器中，这里的数据源可以是来自内存，也可以来自另一个寄存器。之后我们在寄存器上执行一些操作。如果我们对操作的结果关心的话，我们会将操作的结果store在某个地方。这里的目的地可能是内存中的某个地址，也可能是另一个寄存器。这就是通常使用寄存器的方法。
```



# Stack
- 进程的栈区, 主要作用是让函数变得有有组织, 并且能够正常返回
- 栈在逻辑上可以分成好几个栈帧 `stack frame`, 每一个函数调用都会产生一个栈帧![[image 3.avif]]
- 移动通过 `Stack Pointer` 来分配栈空间, 也为新的栈帧分配空间
- 进程的栈空间总是从高地址开始向着低地址使用, 因此分配栈空间是对 `stack pointer` 做减法
- `Return address` 总是会出现在Stack Frame的第一位
- 指向前一个 `Stack Frame` 的指针也会出现在栈中的固定位置
- 关于栈的两个重要寄存器
	- `sp, stack pointer`, 指向整个栈的底部, 代表当前 `stack frame` 底部的位置
	- `fp, frame pointer`, 指向当前 `stack frame` 的顶部



# Struct
