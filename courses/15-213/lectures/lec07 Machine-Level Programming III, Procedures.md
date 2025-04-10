- `procedures` 就是 `functions`, 程序通过以下方法支持函数调用
	- `passing control`, 调用函数时需要执行该函数代码, 返回时回到调用后的代码
	- `passing data`, 函数参数, 返回值的传递
	- `memory management`, 被调用函数可能需要分配一些临时自动变量, 因此需要内存空间来存放这些东西
- ![[Pasted image 20250317152141.png]]

# Stack
- 函数调用通常使用栈来支持, 栈先进先出, 非常符合函数调用这种嵌得越深越早返回的特性
- 一般习惯用 `%rsp` 来存储当前栈顶的内存地址, 同时, 习惯上将栈底安排在高地址, 将栈顶安排在低地址, 因此:
	- 栈分配时就让 `%rsp` 减小, `sub 0x18, %rsp`
	- 栈回收则增大, `add 0x18, %rsp`
- ![[Pasted image 20250317152851.png]]

- `push src`, 可以看成是语法糖, 直接将分配栈空间和移动数据一起做了
- `pop dest`, 也是语法糖


# Passing control
- 两个关键的指令来完成代码执行流 `%rip` 的跳转
	- `call label`, 调用某个函数
	- `ret`, 从 `callee` 返回到 `caller`
- `call label`
	- 会设置好参数 (见下面 passing data)
	- 将 `return address` 放入到栈中, 以便 `callee ret` 可以读到该地址然后跳转回去
	- 将 `%rip` 设置为 `callee` 的地址, 从而将程序执行流跳转到 `callee` 代码中
- `ret`
	- 与 `callee stack frame` 最靠近的 `8-bytes` 就是 `caller` 设置好的 `return address`
	- 调用 `ret` 后会将该地址 `pop` 出来, 并设置为 `%rip` 从而支持返回 (一般来说 `callee` 自己分配的栈空间在 `ret` 之前都自己已经回收了)
- ![[Pasted image 20250317153739.png]]
- ![[Pasted image 20250317153902.png]]
- ![[Pasted image 20250317153913.png]]
- ![[Pasted image 20250317153920.png]]

# Passing data
- `callee` 返回值放在 `%rax`
- `caller` 前六个参数依次放在:
	- `%rdi`
	- `%rsi`
	- `%rdx` 
	- `%rcx`
	- `%r8`
	- `%r9`
	- 其余参数通过栈传递
- ![[Pasted image 20250317154138.png]]

# Managing local data
- 进程的栈只有一个, 然后在每个函数执行的时候为他分配一个栈帧 `stack frame`
- `%rbp, stack base pointer` 存储该函数栈帧底地址
- `%rsp, stack pointer` 存储该函数栈帧顶地址
- 函数可以自由地通过移动 `%rsp` 来分配回收局部变量, 他们在函数返回时都会被回收掉
- 但是操作系统为进程栈预留的空间是有限的, 因此局部变量不应该太大, 函数调用也不应该太深, 不然会导致栈溢出
- ![[Pasted image 20250317154813.png]]
- ![[Pasted image 20250317154852.png]]

# Register saving conventions
- 寄存器也是不能随便乱用的, 有些可能存储着 `caller` 的数据, 用完要将数据复原回去
- 因此引入两种保护机制
	- `caller save`, `caller` 预设 `callee` 可能会修改该寄存器的数据, 因此在调用之前就先将值保持在自己的栈帧内, 调用后再复原到寄存器中
	- `callee save`, `callee` 预设 `caller` 可能需要改寄存器的数据, 因此在使用之前先将该数据保持在自己的栈帧内, 返回之前再复原到寄存器中
- `caller save`
	- `%rax`, 返回值
	- `%rdi, %rsi, %rdx, %rcx, %r8, %r9`, 函数参数
	- `%r10, %r11`, 临时使用的寄存器
- `callee save`
	- `%rbx, %r12, %r13, %r14`, 临时使用的寄存器
	- `%rbp`, 栈底指针
	- `%rsp`, 栈顶指针

# Recursion
- 有了函数调用栈道模型之后, 递归就很好理解了
- 一个函数可以有多个调用实例
- 他们的数据不用显式保护起来, 函数调用栈做了大部分工作
- 将递归改为非递归也比较容易想象了

# Summary
![[Pasted image 20250317155834.png]]
