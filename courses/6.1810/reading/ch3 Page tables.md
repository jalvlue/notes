- `os` 为每个进程提供私有的地址空间和内存
- 页表决定了每块内存的意义, 以及实际物理内存的访问权限
- 从而让保证了进程的隔离性同时对一块物理内存进行复用
- xv6 使用虚拟内存的tricks
	- 将同一块物理内存映射到多个虚拟内存上 (trampoline page)
	- 使用一个未映射的 page 保护内核栈和用户栈 (防止栈溢出)

# 3.1 paging hardware
- `riscv` 的指令 (包括用户空间指令和内核指令) 都通过虚拟内存对实际物理内存进行操作
- Xv 6 在 Sv 39 RICS-V 上运行, 虚拟地址的高 25 位不使用, 只是用后 39 位
	- 后 39 中的后 12 位作为页内 offset (`12-bit, 4096Byte` 也就是一个页的大小)
	- 前 27 位作为页表该页在 `page table` 上的 `index`
- 单级页表描述: 页表一共长 `2^27` (有 `2^27` 个 `page table entry - PTE`) 每一个页表项共由 56 个有效位组成, 以供虚拟内存的 `27` 位 `index` 进行索引
	- 44 位实际物理地址
	- 10 位控制 flags
![[Pasted image 20240518164104.png]]
- 在上图中可以看出, 虚拟内存有高 `25-bit` 的空闲空间, 对于实际物理内存, 也有 `10-bit` 的空闲空间
- Sv 39 RISC-V 已经相当够用了, 此外还有 Sv 48 RISC-V
- Xv 6 采用三级页表
	- 页表作为 `paging hardware`, 他们的地址都是实际的物理地址
	- 一级页表的 PPN 存储着 n 多二级页表的起始地址
	- 二级页表的 PPN 存储着 n 多三级也表达起始地址
	- 三级页表的 PPN 则就是实际物理内存空间的 PPN
	- 省空间: 如果某一个高级页表的某个 `entry` 是 ` invalid ` 的, 此时 os 就不用给这个 `entry` 的子页表分配空间
	- 缺点: 但是 CPU 需要多次查询(使用 TLB(Translation Look-aside Buffer, 快表缓存减轻))
![[Pasted image 20240518164723.png]]
- 页权限 flags
	- V
	- R
	- W
	- E
	- ...
- 进程要使用页表时, 需要将它的根页表地址写进 CPU 的 `satp` 寄存器中 (实际上发生在上下文切换的过程中), 然后 CPU 就可以通过这个根页表一步一步查询该进程虚拟内存对应的物理地址
- 每个 CPU 有自己独立的 `satp` 寄存器

# 3.2 kernel address space
![[Pasted image 20240518171126.png]]
- 内核内存分布, 内核同样也是使用虚拟内存, 但是对于一些特殊的使用直接映射, 也就是虚拟内存和实际物理内存相同
	- 一些设备 `0x0~0x80000000`
	- RAM, `0x80000000~0x88000000`, 整个 `riscv` 物理内存都直接映射进了内核的虚拟地址空间
	- 直接映射简化了内核访问物理内存空间的过程
- 此外, 内核虚拟内存空间除了直接映射块之外, 还有大片大片的非直接映射映射, 用于各种目的
	- `trampoline page`
	- `kernel stack pages`
- `kernel/memlayout.h` 文件定义了上图的很多关键地址
```c
// Physical memory layout

// qemu -machine virt is set up like this,
// based on qemu's hw/riscv/virt.c:
//
// 00001000 -- boot ROM, provided by qemu
// 02000000 -- CLINT
// 0C000000 -- PLIC
// 10000000 -- uart0 
// 10001000 -- virtio disk 
// 80000000 -- boot ROM jumps here in machine mode
//             -kernel loads the kernel here
// unused RAM after 80000000.

// the kernel uses physical memory thus:
// 80000000 -- entry.S, then kernel text and data
// end -- start of kernel page allocation area
// PHYSTOP -- end RAM used by the kernel

// qemu puts UART registers here in physical memory.
#define UART0 0x10000000L
#define UART0_IRQ 10

// virtio mmio interface
#define VIRTIO0 0x10001000
#define VIRTIO0_IRQ 1

// core local interruptor (CLINT), which contains the timer.
#define CLINT 0x2000000L
#define CLINT_MTIMECMP(hartid) (CLINT + 0x4000 + 8*(hartid))
#define CLINT_MTIME (CLINT + 0xBFF8) // cycles since boot.

// qemu puts platform-level interrupt controller (PLIC) here.
#define PLIC 0x0c000000L
#define PLIC_PRIORITY (PLIC + 0x0)
#define PLIC_PENDING (PLIC + 0x1000)
#define PLIC_SENABLE(hart) (PLIC + 0x2080 + (hart)*0x100)
#define PLIC_SPRIORITY(hart) (PLIC + 0x201000 + (hart)*0x2000)
#define PLIC_SCLAIM(hart) (PLIC + 0x201004 + (hart)*0x2000)

// the kernel expects there to be RAM
// for use by the kernel and user pages
// from physical address 0x80000000 to PHYSTOP.
#define KERNBASE 0x80000000L
#define PHYSTOP (KERNBASE + 128*1024*1024)

// map the trampoline page to the highest address,
// in both user and kernel space.
#define TRAMPOLINE (MAXVA - PGSIZE)

// map kernel stacks beneath the trampoline,
// each surrounded by invalid guard pages.
#define KSTACK(p) (TRAMPOLINE - ((p)+1)* 2*PGSIZE)

// User memory layout.
// Address zero first:
//   text
//   original data and bss
//   fixed-size stack
//   expandable heap
//   ...
//   TRAPFRAME (p->trapframe, used by the trampoline)
//   TRAMPOLINE (the same page as in the kernel)
#define TRAPFRAME (TRAMPOLINE - PGSIZE)

```
- `xv6 - riscv` 中, 使用的 RAM 物理内存地址是 `0x80000000 ~ 0x88000000` 一共 `128MB`
- 上面的 `memlayout.h` 文件也定义了很多

# 3.3 code: creating an address space
- `pagetable_t`, 指向一个 RISC-V 根页表, 页表的核心数据结构
- TODO: 读代码