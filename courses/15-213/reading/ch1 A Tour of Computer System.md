- `computer system = hardware + system softwares`
- 架构易变, 思想永存
```c
#include <stdio.h>

int main() {
	printf("hello, world\n");
	return 0;
}
```
![[Pasted image 20241218214603.png]]
- `hello.c` 就是一个普通的 `ascii text`, 计算机系统可以将他翻译为一个可执行程序, 并执行它, 发挥它的作用

# 1.1 information is bits+context
- `source file, ascii text, encoded as byte sequence, one byte for a character`
- `binary file`
- 一切皆文件, 一切文件皆字节流, 影响其意义的除了字节流中的每一个 `bit`, 还有它的背景 `context`, 一个字节序列可以被解释为整数, 浮点数, 代码... 根据不同的背景有不同的含义
- 了解整数, 浮点数在计算机世界是如何表示的很重要 (区分于现实世界的数学概念)


# 1.2 programs are translated by other programs into different forms
- `hello.c` 是人类可读的 `source file`
- 通过编译器翻译为 `machine language instructions binary object file`
- 再把多个 `object` 链接打包为一个 `executable binary object file`, 存储在磁盘中
- 然后就可以被计算机系统加载并运行
- `gcc` 是 `unix-like system` 最常见的 c 编译器
- 一个典型的编译过程
![[Pasted image 20241218230700.png]]
- `gcc -o hello hello.c`
	- `source text`
	- `assembly text`
	- `relocatable object file`
	- `executable object file`
	- `preprocessor, cpp`, 预处理, 处理带 `#` 的代码, 其中的处理只是做简单的宏展开或者库引入, 例如 `#include <stdio.h>` 就是直接将标准 io 中的代码直接插入到 `.c` 源文件中, 生成预处理后的 `.i ` 文件
	- `compiler, cc1`, 编译阶段, 最核心的一个部分, 软硬件之间交互的唯一途径, 不同高级语言的编译器在编译程序之后都会落到汇编语言中
	- `assembler, as`, 汇编阶段, 将汇编代码转为对应的机器码编码, 汇编语言是文本, 人类可读的, 这里的作用就是将其中每一个指令都编码成二进制机器指令的格式, 生成的是 `relocatable object binary file`
	- `linker, ld`, 链接阶段, 一个程序可能涉及到多个源文件, 连接器就是将这些源文件生成的 `relocatable object files` 链接成为一个完整的 `executable object binary file`, 例如 `printf` 是一个预先编译好的 `object file`, 这里就是将它和 `hello.o` 链接在一起
- 这些过程以及其中的组件构成了整个 c 编译系统


# 1.3 it pays to understand how compilation system works
- 了解编译系统还是比较重要的
- `optimizing program performance`, 了解不同的 c 风格程序会被转化为怎样的机器代码有助于写出更高效的程序, 优化执行效率
- `understanding link-time errors`, 链接错误是比较常见的, 了解编译系统对和链接过程理解链接错误
- `avoiding security holes`, 很多漏洞来自细小的底层细节, 理解底层才能更好规避有漏洞的程序


# 1.4 processors read and interpret instructions stored in memory
- 操作系统内核以及其外围为程序和处理器之间打通了关系, 处理器可以直接加载应用程序被处理器执行

### 1.4.1 hardware organization
![[Pasted image 20241219201505.png]]
- `buses`, 总线, 作用是在各个组件之间传递数据 (代码也是数据), 上面提到数据就是字节流, 总线为了高效会每次传输固定大小的数据, 这个大小是一个系统参数, 成为字 `word size`, 一般来说是 `4-byte, 32-bit` 或者 `8-byte, 64-bit`
- `io devices`, io 设备, 计算机系统核心与外界交互的途径, 正如名字一样就是接受输入, 并将结果输出返回的设备, 常见的有键盘鼠标, 磁盘, 网络适配器..., io 设备通过 `controller/adapter` 连接到 `io bus`
- `main memory`, 内存, 临时存储, 存储程序运行过程中的临时数据, 在程序的视角中就是一个巨大的字节数组, 其中每一个字节都有自己的编号 (地址), 因此是以随机存取的, 通常由 `DRAM, dynamic random access memory` 构成
- `processor`, cpu 处理器
	- `word-size storage device, registers`, 通用寄存器组
	- `program counter register`, 程序计数器, 特殊的寄存器, 里面存放着下一个将要执行的指令在内存中的地址, 每一个指令的长度为 1 或多个字节, 
	- `ALU, arithmetic and logic unit`, 算术逻辑单元


### 1.4.2 running the `hello` program
![[Pasted image 20241219203426.png]]
- 用户告诉操作系统及其外围他将执行 `hello` 程序

![[Pasted image 20241219203614.png]]
- 系统将应用程序的数据和代码都加载到内存中

![[Pasted image 20241219203748.png]]
- cpu 的 `pc` 跳到应用程序代码开始地址, cpu 开始执行应用程序

# 1.5 caches matter
- 磁盘->内存->寄存器->cpu 执行, 为了执行应用程序计算机系统做了数据传输的工作, io 比 cpu 计算是慢得多的, 因此太多太频繁的 io 会拖慢程序的运行, 系统应该尽可能的减少数据的传输
- 目前的物理存储设备上, 存储容量大的速度慢, 存储容量小速度快的造价贵, 同时 cpu 速度和存储设备 io 的速度差距越来越大
- 因此使用 `cache` 缓存的思想非常重要, 在主存和寄存器之间再加大小速度适中的 `cache`, 并根据空间/时间局部性对编译器做优化, 尽量提高缓存命中率
	- 多级缓存
	- `SRAM, static random access memory`, 静态随机存取内存, 原理跟 `DRAM` 不同, 适合做缓存或寄存器


# 1.6 storage devices form a hierarchy
- 缓存是一种思想, 在存储系统的各级都可以利用这种思想提高程序效率
- 存储系统层级金字塔中, 每一个设备都是其下一级的缓存
![[Pasted image 20241219205005.png]]


# 1.7 the operating system manages the hardware
- 操作系统具有最高权限, 管理着计算机硬件
- 所有应用程序访问硬件资源都必须通过操作系统, 而不能直接使用
	- 为了包含硬件不被恶意应用程序滥用
	- 为了向应用程序提供统一的访问接口, 屏蔽不同设备之间的差异
- 操作系统为了达成这些目标做了许多努力
	- `process`
	- `virtual memory`
	- `file`
	- `syscall`

![[Pasted image 20241219205659.png]]

### 1.7.1 processes
- 进程, 是操作系统对 cpu 的抽象, 每一个需要占用 cpu 资源, 在计算机系统上运行的程序都是一个进程
- 进程实际上是为应用程序创造了一种假象: 整个计算机上只有它自己一个程序, cpu 只为它服务, 不断地执行它的指令, 没有中断
- 有了进程抽象, 处理器可以在操作系统的指挥下间断交织地执行多个程序的代码, 从而实现 `concurrency`
- `context switch`, 上下文切换, 所谓上下文就是程序被中断之后, 要恢复回继续执行所需要的状态, 通常包括 `pc, registers`
![[Pasted image 20241219210821.png]]

### 1.7.2 threads
- 现代 cpu 单核能力已经很高了, 同时单核 cpu 对一些 io/data 密集应用不太够用, 因此发展出了多核处理器
- 应用程序可以通过 `threads` 线程来获得多核 cpu 的支持, 线程就是执行流, 有自己的上下文, 但是共享代码和全局数据

### 1.7.3 virtual memory
- 虚拟内存将主存抽象为一个连续不断, 独有的字节数组给进程, 也是一种假象, 进程认为整个计算机系统的内存都是它的, 全都放置着它的数据
- 虚拟内存允许操作系统玩一些花活 (共享内存页, copy on write...)
- 程序的虚拟内存视角
	- `program code and data`
	- `heap`
	- `shared libraries`
	- `stack`
	- `kernel virtual memory`

![[Pasted image 20241219212330.png]]

### 1.7.4 files
- 文件就是字节流
- 所有的 io 设备都被操作系统抽象为了文件, 从而为应用程序提供了统一的 api
- 因此应用程序向访问什么 io 设备, 都可以直接使用文件 api, 读取或写入一段字节流

# 1.8 systems communicate with other systems using networks
- 网络是计算机发明之后又一伟大的发明, 让计算机之间可以远程传递信息
- 网络接口也被操作系统抽象为了 io 设备
![[Pasted image 20241219213049.png]]
![[Pasted image 20241219213109.png]]


# 1.9 important themes
- `computer system = hardware + system software`

### 1.9.1 Amdahl's law
- 将整个计算机系统组件化, 判断某些组件效率提高之后可以给整个系统带来怎样的效率提升
![[Pasted image 20241219213414.png]]

### 1.9.2 concurrency and parallelism
- `concurrency` 并发, 是从单核 cpu 的角度出发的, 它可以利用中断快速地交织执行多个程序, 从而带来多个程序同时在运行的假象
- `parallelism` 并行, 是从多核 cpu 的角度出发的, 多个程序 (线程) 在不同的 cpu 核心上在同一时刻一起执行
- `thread-level concurrency` 线程级并发, 利用现代多核 cpu 和超线程技术
![[Pasted image 20241219214136.png]]
![[Pasted image 20241219214157.png]]
- `instruction-level parallelism` 指令级并行, 根据执行一条指令的不同阶段构建多级流水线
- `single-instruction, multiple-data(SIMD) parallelism`, SIMD, 特殊的加速硬件

### 1.9.3 the importance of abstractions in computer science
- 抽象是庞大计算机世界最根本的东西, 计算机系统为了易用性已经做了很多非常有效, 优雅的抽象, 这对应用程序编程也有借鉴意义, 例如 api 设计, 设计简单易用易于理解的 api
- 甚至计算机系统本身也可以被抽象出来
![[Pasted image 20241219215259.png]]


# 1.10 summary
```ad-summary
A computer system consists of hardware and systems software that cooperate
to run application programs. Information inside the computer is represented as groups of bits that are interpreted in different ways, depending on the context. Programs are translated by other programs into different forms, beginning as ASCII text and then translated by compilers and linkers into binary executable files.

Processors read and interpret binary instructions that are stored in main mem-ory. Since computers spend most of their time copying data between memory, I/O devices, and the CPU registers, the storage devices in a system are arranged in a hi-erarchy, with the CPU registers at the top, followed by multiple levels of hardware cache memories, DRAM main memory, and disk storage. Storage devices that are higher in the hierarchy are faster and more costly per bit than those lower in the hierarchy. Storage devices that are higher in the hierarchy serve as caches for de-vices that are lower in the hierarchy. Programmers can optimize the performance of their C programs by understanding and exploiting the memory hierarchy.

The operating system kernel serves as an intermediary between the applica-tion and the hardware. It provides three fundamental abstractions: (1) Files are abstractions for I/O devices. (2) Virtual memory is an abstraction for both main memory and disks. (3) Processes are abstractions for the processor, main memory, and I/O devices.

Finally, networks provide ways for computer systems to communicate with one another. From the viewpoint of a particular system, the network is just another I/O device.
```
