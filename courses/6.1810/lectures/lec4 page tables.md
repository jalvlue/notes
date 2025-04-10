[[ch3 Page tables]]

# Overview
- 虚拟内存-物理内存之间的映射
- 通过虚拟内存给每个进程提供拥有完整地址空间的假象
- 通过虚拟内存对进程隐藏实际物理内存, 让实际物理内存操作都交由 kernel 来做, 从而提供进程的隔离性
- 地址空间, `address spaces`
- `riscv` 中支持虚拟内存的硬件
- 内核地址空间+用户地址空间结构
- `xv6` 中管理虚拟内存的具体代码实现


# Address Spaces
- 让每一个进程都以为他们有自己独立的地址空间, 从而让这些进程不能引用任何不属于自己的内存![[attachment/image 2.avif]]
- 从而达到强隔离性


# Page Tables
- 页表是进行 `va-pa` 转换的常见方法
- 在 CPU 的视角中, 任何一条带有地址的指令 `sd $7, (a0)`, 其中的地址都是虚拟内存地址, 需要转到 ` MMU, memory management unit ` 中进行转换![[image 1 1.avif]]
- 每一个 CPU 都有一个自己独有的 `satp` 寄存器, 保存当前进程的根页表在物理内存中的地址, 然后 MMU 就会根据这个地址去找到页表并进行转换 [[ch3 Page tables#3.1 paging hardware]]
- 页表过大:
	- 分页
	- 分级
- PTE flags: 10-bits
	- `Valid`
	- `Readable`
	- `Writable`
	- `eXecutable`
	- `User`
	- ...

# Translation Lookaside Buffer
- `TLB`, 俗成快表, 是 PTE 的缓存
- `TLB` 存在于 CPU 中, CPU 进行 `va-pa` 转换时会先读缓存
- 这一切都由硬件实现, 对 OS 透明
- 因此不需要太过关心


# Kernel Page Table
![[attachment/image.avif]]
- 物理地址大于 `KERNBASE, 0x8000,0000` 则会走向 DRAM, 小于则走向各种 IO 设备
- 地址规则完全是由硬件设计者决定的
- 主板上电之后, 第一件事就是运行存储在 `0x1000` 的 `boot ROM` 中的代码
- 完成之后会跳到 `0x8000,0000` (硬件决定的), 因此 OS 需要保证此时这里有指令能接着启动 OS [[ch2 Operating system organization#2.6 Code starting xv6, the first process and system call]]
- 在内核的虚拟内存空间中, 绝大部分跟实际物理地址空间都是平等映射, 只有 `Trampoline, Kstack` 是例外
	- 这是因为在虚拟内存处使用了 `guard page` 来方式栈溢出, 但是又不想浪费物理地址空间, 因此就采取在靠后的虚拟地址映射
- 此外内核的虚拟地址空间对应的 page table 也设置了页面的权限, 以防内核本身的 bug 导致问题

```ad-faq
学生提问：对于不同的进程会有不同的kernel stack吗？

Frans教授：答案是的。每一个用户进程都有一个对应的kernel stack

学生提问：用户程序的虚拟内存会映射到未使用的物理地址空间吗？

Frans教授：在kernel page table中，有一段Free Memory，它对应了物理内存中的一段地址。XV6使用这段free memory来存放用户进程的page table，text和data。如果我们运行了非常多的用户进程，某个时间点我们会耗尽这段内存，这个时候fork或者exec会返回错误。

同一个学生提问：这就意味着，用户进程的虚拟地址空间会比内核的虚拟地址空间小的多，是吗？

Frans教授：本质上来说，两边的虚拟地址空间大小是一样的。但是用户进程的虚拟地址空间使用率会更低。

学生提问：如果多个进程都将内存映射到了同一个物理位置，这里会优化合并到同一个地址吗？

Frans教授：XV6不会做这样的事情，但是page table实验中有一部分就是做这个事情。真正的操作系统会做这样的工作。当你们完成了page table实验，你们就会对这些内容更加了解。
```

- 用户进程空间: ![[attachment/image 1.avif]]

# code
- 有关虚拟内存的代码都在 `kernel/vm.c` 中
- `kernel/main.c` 在初始化 CPU 时调用了 `kvminit(), kvminithart()` 这两个函数, 对内核的虚拟内存进行初始化


### kvminit()
![[Pasted image 20240723233750.png]]
![[Pasted image 20240723233805.png]]
- 首先, 给内核根页表, 全局指针变量 `kernel_pagetable` 使用 `kalloc()` 分配了一个 `page`, 并将内容全置为 `0`, 表示二级页表还没有建立起来
- `kvmmap() -> mappages()` ![[Pasted image 20240723234100.png]]
- 这里调用了 `walk(pagetable, a, alloc=1)` 进行二级三级页表的搭建 (如果没有存在), 然后将该物理地址写入到三级页表对应 index 的 `pte` 中, 并设置标识位, `line 165`, 此时这个 pte 是 `valid` 的, 因此新增了一个 `va page - pa page` 的映射


### kvminithart()
![[Pasted image 20240723235010.png]]
- `kvminithart` 主要是将设置好的 `kernel_pagetable` 写入到当前 CPU 的 `satp` 寄存器中
- 此后开启了新世界, CPU 指令中的地址在此之前都是实际物理地址, 此后全都是需要转换虚拟地址 (虽然内核地址空间的绝大部分都是使用直接映射, 转换后 `va==pa`)

# walk()
![[Pasted image 20240723235301.png]]
- 建立 `page table` 时, `alloc=1`, 查询时 `alloc=0`
- 主要就是根据 `va` 和当前级别的 `pagetable` 抽离出存储下一级信息的 `pte` 然后一级一级处理, 最后返回第三季页表中的与 `va` 对应的那个 pte
- 如果是建立则可以新分配
