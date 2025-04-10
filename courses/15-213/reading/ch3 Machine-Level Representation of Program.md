- 机器执行 `machine code`, 也就是二进制指令, 这些低级指令包括操作数据, 操作内存, 读写设备
- 这些指令通常是操作系统和计算机硬件共同商议好的, 称为 `ISA, instruction set architecture`, 编译器在这基础之上, 将高级语言编写的复杂逻辑翻译成这样的一个个简单指令
- `gcc` 首先会将 c 程序编译成汇编文本程序 `textual assembly code`, 然后通过汇编器汇编成二进制 `binary assembly code`, 最后通过链接器生成可执行二进制程序 `binary executable`, 其中 `binary code` 就是 `machine code`
- 高级语言和编译器有很多的好处, 跨平台, 编译器类型检查, 比起直接编写汇编有很多的优势
- 了解程序运行的关键是看懂高级语言编译出来实际在机器上运行的汇编代码, 了解常见的汇编指令的运行效率
	- 优化程序的执行顺序
	- 去除不必要的计算
	- 将耗时的操作改为高效的操作指令
	- 甚至将递归操作改为迭代操作


# 3.1 a historical perspective
- `intel x86` 是处理器产品线, 最新的是 `x86-64`
- `intel` 每一代处理器以及指令集为了运行前代软件做了许多兼容工作, 很多指令都是由于历史原因而加入的


# 3.2 program encodings
- 两个源文件 `p1.c, p2.c`
- `linux> gcc -Og -o p p1.c p2.c`
	- 使用 `gnu c compiler` 来编译 c 程序
	- 优化等级为 `-Og`, 表示只启用很少的优化, 让编译后的汇编代码尽可能遵从原来 c 程序的结构
- `gcc` 是一个工具链合集, 其中按照各个工具的顺序分为了
	- `c preprocessor, cpp` 预处理器主要做的是展开带 `#` 的内容, 例如宏展开或者头文件展开, 此时做的只是简单的文本替换, 生成 `*.i` 文本文件
	- `compiler` 编译器将预处理后的程序编译成汇编代码, 此时生成 `*.s` 汇编文本文件, 此时就是实际上在机器运行的文本指令格式了
	- `assembler` 汇编器将汇编文本代码转换成二进制对象文件, 此时就是在机器上实际运行的二进制指令格式了 `*.o`
	- `linker` 链接器将源文件的对象文件和依赖的库文件链接打包在一起, 并确定他们的地址, 然后生成一个可执行程序 `-o p`

### 3.2.1 machine-level code
- 计算机系统是建立在层层抽象之上的, 机器级代码也依赖一些更加底层的抽象
	- `ISA`, 指令集, 指令集包含了计算机系统可以提供的一切指令, 机器级代码就是利用这些指令来构建复杂的应用, `ISA` 抽象提供了所有指令都是顺序执行的保证, 同时, `ISA` 也是一种表示处理器状态的方式, 不同的指令可以修改处理器的状态
	- `virtual memory`, 虚拟内存提供的抽象让程序认为自己拥有一块连续的字节数组内存空间, 让应用程序的编写变得简单
- 对于寄存器的使用是机器级代码和高级语言代码之间最大的区别
	- `pc, %rip, program counter`, 程序计数器, 存储下一个将要执行的指令在内存中的地址
	- `integer register, %r`, 整数计数器, 可以存放 `64-bits` 的信息, 这些寄存器是做算数和逻辑运算存放数据以及函数调用, 重要的部分
	- `condition code registers`, 状态寄存器, 通常存放上次算数/逻辑运算之后得到的状态, 在 `if, while` 语句的实现中非常重要
	- `vector registers`
- 在机器级代码中, 程序的数据只是一大堆在内存中的字节序列, 没有数据类型的区分, 有的只是从某个位置读取 n 字节的数据, 对这些数据做一些处理
- 数据类型, 类型检查, 越界检查, 以及实际使用什么指令做何种操作则是编译器的职责
- 每个机器级指令的功能是非常简单的, 只有在编译器的组织下, 一大段指令相互配合才能发挥出巨大的作用

### 3.2.2 code examples
```c
/* $begin 010-mstore-c */
long mult2(long, long);

void multstore(long x, long y, long *dest) {
  long t = mult2(x, y);
  *dest = t;
}
/* $end 010-mstore-c */

```
```asm
010-mstore.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <multstore>:
   0:   f3 0f 1e fa             endbr64
   4:   53                      push   %rbx
   5:   48 89 d3                mov    %rdx,%rbx
   8:   e8 00 00 00 00          call   d <multstore+0xd>
   d:   48 89 03                mov    %rax,(%rbx)
  10:   5b                      pop    %rbx
  11:   c3                      ret
```
- `gcc -Og -S mstore.c`
- 在反汇编视角中, 可以看到指令与其二进制编码之间一一对应的关系
- 指令是变长的, 也是数据, 会被加载到内存中
- 单对象文件, 非链接后可执行文件, 此时的反汇编代码中, 可以看到函数 `mulstore` 地址为 0, 此时是独立存在的, 因此也成为可重定位对象二进制文件


```c
/* $begin 010-multmain-c */
#include <stdio.h>

void multstore(long, long, long *);

int main() {
  long d;
  multstore(2, 3, &d);
  printf("2 * 3 --> %ld\n", d);
  return 0;
}

long mult2(long a, long b) {
  long s = a * b;
  return s;
}
/* $end 010-multmain-c */

```
```asm
00000000000011cc <mult2>:
    11cc:       f3 0f 1e fa             endbr64
    11d0:       48 89 f8                mov    %rdi,%rax
    11d3:       48 0f af c6             imul   %rsi,%rax
    11d7:       c3                      ret

00000000000011d8 <multstore>:
    11d8:       f3 0f 1e fa             endbr64
    11dc:       53                      push   %rbx
    11dd:       48 89 d3                mov    %rdx,%rbx
    11e0:       e8 e7 ff ff ff          call   11cc <mult2>
    11e5:       48 89 03                mov    %rax,(%rbx)
    11e8:       5b                      pop    %rbx
    11e9:       c3                      ret
```
- `gcc -Og -o proc main.c mstore.c`
- 此时生成的是链接后的, 完整的可执行程序
- 可以看到, `mult2, mulstore` 函数都被重定位了, 
- `mulstore` 函数的指令跟单对象文件中的内容几乎一样, 只有 `call mult2` 时, 链接器将对应调用函数的地址填充了上去


### 3.2.3 notes on formatting
- Skip
- 很多时候在非常底层的地方甚至连 C 语言都无法处理, 此时需要同时编写 C 语言和汇编
	- 使用汇编编写完整的函数, 然后在链接阶段与 C 语言函数链接起来
	- 使用 `gcc` 提供的特性在 C 语言中嵌入执行汇编指令


# 3.3 data formats
- 由于历史原因, `Intel` 将 
	- `8` 个二进制位一组称为 `byte`
	- `16` 个二进制位一组称为 `word`
	- `32` 个二进制位一组称为 `double word`
	- `64` 个二进制位一组称为 `quad word`
![[Pasted image 20250106213451.png]]


# 3.4 accessing information
- 各种历史原因导致现在最新的 `x86-64` 处理器有 `16` 个通用寄存器, `%rax~%rbp, %r8~%r15`
- 通用寄存器就是有多种灵活用途的寄存器, 其中特别的是存储当前 cpu 进程栈指针的 `%rsp`
- 为了兼容前代 `32, 16, 8` 位 cpu 以及程序, 这些寄存器也可以使用部分的位, 从而得到兼容效果
![[Pasted image 20250107194506.png]]
- 这些通用寄存器和他们的常见用途以及在函数调用中的行为

### 3.4.1 operand specifier
- 大多数的指令需要参数, 具体地, 需要数据源并存储指令结果
	- `src`, 源操作数
		- 可以是某个寄存器中存储着的值 `R[ra]`
		- 可以是内存中某个地址后连续的 `b bytes` 存储的值 `Mb[Addr], Imm(rb, ri, s) = Imm + R[rb] + R[ri]*s`
		- 也可以是某个常量 (立即数) `Imm`
	- `dst`, 目的操作数
		- 可以是寄存器, 将结果存储到该寄存器中
		- 可以是内存地址, 将结果存储到内存内该地址后连续 `b bytes`
- 内存寻址是比较复杂的, 这是为了数组或者结构体这种聚合数据类型的访问方便设计的

### 3.4.2 data movement instructions


# 3.5 arithmetic and logical operations


# 3.6 control


# 3.7 procedures


# 3.8 array allocation and and access


# 3.9 heterogeneous data structure


# 3.10 combining control and data in machine-level programs


# 3.11 floating point code


# 3.12 summary
