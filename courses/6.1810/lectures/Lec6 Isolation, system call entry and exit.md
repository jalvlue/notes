[[ch4]]


# 6.1 trap 机制
- 用户程序运行时, 在什么情况下会发生用户空间和内核空间之间的切换
	- 程序执行系统调用
	- 程序运行时出现 `page fault`, 运算时除以 0 等错误或异常, 需要内核介入处理现场
	- 一个设备触发了中断, 使得当前程序需要运行响应对应的内核设备驱动
- 用户空间和内核空间之间的切换被称为 `trap`
	- `trap` 处理这一切程序运行过程中从用户权限到内核权限的变化
	- 这个过程中硬件的状态非常重要, `trap` 的很多工作都是将硬件从适合用户程序的 user 状态切换到适合运行内核代码的 supervisor 状态
- 32 个用户寄存器 (用户程序可读写使用的寄存器)
- 一些重要的寄存器, 这些寄存器表明了计算机/CPU 在执行系统调用时的状态
	- `PC, Program Counter register`, 程序计数器, 里面存放下一条要执行的指令在内存中的地址
	- `mode signal bit`, 一个标志位 (??? 是在哪个寄存器中吗), 标识当前 CPU 运行在 `user mode / supervisor mode`
	- `SATP, Supervisor Address Translation and Protection register`, 包含当前运行的进程的 `root page table` 在内存中的地址
	- `STVEC, Supervisor Trap Vector Base Address register`, 指向内核中, 处理 trap 指令的起始地址
	- `SEPC, Supervisor Exception Program Counter register`, 在 trap 的过程中保存程序计数器的值
	- `SSCRATCH, supervisor scratch register`, TODO
- 
- `trap` 流程: 最开始时, CPU 处于用户态, 需要对 CPU 状态寄存器和用户进程信息进行一些处理才能在返回时继续执行用户的程序, 让用户感知不到 `trap`
		1. 保存 32 个用户寄存器的值
		2. 保存 `PC` 寄存器
		3. 将 `CPU mode signal bit` 改为 `supervisor mode`
		4. 将 `SATP` 寄存器的值从 `user root page table address` 改为 `kernel root page table address`
		5. 将 `stack pointer` 指向内核的一个地址, 从而内核可以使用这个栈进行 C 函数调用
		6. 跳入内核的 C 代码
		7. 执行内核代码
- `trap` 是 os 安全和隔离的重要部分, 不应该让任何的用户代码参与或者依赖用户进程信息, 因此所有的用户状态都只是保存, 而不会读取使用
- `trap` 应该对用户是透明的, 用户不应该感知到任何的机制或执行过程
- `mode signal bit`: 标志位设置为 `supervisor mode` 时, CPU 多了一些权限:
	1. 可以读写 CPU 控制/状态寄存器 `SATP, STVEC, SEPC, SSCRATCH...`
	2. 可以使用 `PTE_U == 0` 的页表, 
		- `PTE_U == 1`: 表明用户代码可以使用这个页表
		- `PTE_U == 0`: 只有 `supervisor mode` 可以使用这个页表
- CPU 的 `supervisor mode` 可以说是对应 `os kernel mode`, 也就是还是不能任意读写物理地址, 还是要使用 `kernel page table` 来通过虚拟内存访问实际物理内存


# 6.2 Trap 代码执行流程
- `trap` 调用链 (以 `syscall-write` 触发的为例)
	1. 用户代码调用 `write(...)` 函数
	2. `write()` 执行 `ecall` 来执行系统调用, `ecall` 将 CPU 切换到 `supervisor mode`, 保存用户进程状态, pc 指向 `trampoline.s@uservec`, `uservec()` 函数就是在内核态第一个执行的程序
	3. 