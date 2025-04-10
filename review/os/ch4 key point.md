# 4.1 什么要有虚拟内存
- 操作系统为每一个进程安排一片虚拟内存空间
- 虚拟内存地址如何落到实际物理内存地址对进程是透明的, 由操作系统管理
- **操作系统会提供一种机制，将不同进程的虚拟地址和不同内存的物理地址映射起来。**
- 进程的虚拟地址通过 CPU 中的 MMU (memory manage unit)转换为物理地址

- 内存分段: 根据进程内存由几个逻辑分段组成的原理, 将虚拟内存按段分配
	- 代码段
	- 数据段
	- BSS 段
	- ...
- 通过选择因子 (段编号) + 段内偏移获得某一段内的某一个虚拟地址
![[Pasted image 20240518162126.png]]
![[Pasted image 20240518162203.png]]
- 分段可能会导致内存外部碎片






# 4.6 深入理解 Linux 虚拟内存管理
- 用户态虚拟内存空间
	- 数据段: 存放的是有初始值的全局变量和静态变量
	- BSS 段: 存放未指定初始值的全局变量和静态变量, 加载进内存后会初始化为 0 值
	- ==TODO== 搞明白静态变量
	- 文件映射与匿名映射区: 动态链接库 + `mmap` 系统调用映射的文件
- 堆有一个 `brk` 指针, 使用 `malloc-brk` 函数+系统调用时, brk 指针会从低地址往高地址移动, 拓展堆
- 堆之上有一块待分配空间, 再往上就是文件映射与匿名映射区, 增长是从高地址往低地址, 使用 `malloc-mmap` 函数+系统调用扩展
- 再往上是为栈留的待分配空间
- 栈最重要的就是栈顶指针 (使用 `RSP` 寄存器保存), 和栈基指针 (使用 `RBP` 寄存器保存)

- 64 位机器寻址范围可达到 `2^64`, 但目前只是用了 `48` 位, 形成了空洞
![[Pasted image 20240515155902.png]]

- `fork` 系统调用会创造新的进程, 父进程的虚拟内存空间, 页表都是直接拷贝一份, 
- 而 `vfork` 则会创建一个共享父进程虚拟内存空间的子进程 (就像是线程), 他会引用父进程的虚拟内存空间, 从而共享页表和虚拟内存中的数据

```c
struct mm_struct {
    unsigned long task_size;    /* size of task vm space */
    unsigned long start_code, end_code, start_data, end_data;
    unsigned long start_brk, brk, start_stack;
    unsigned long arg_start, arg_end, env_start, env_end;
    unsigned long mmap_base;  /* base of mmap area */
    unsigned long total_vm;    /* Total pages mapped */
    unsigned long locked_vm;  /* Pages that have PG_mlocked set */
    unsigned long pinned_vm;  /* Refcount permanently increased */
    unsigned long data_vm;    /* VM_WRITE & ~VM_SHARED & ~VM_STACK */
    unsigned long exec_vm;    /* VM_EXEC & ~VM_WRITE & ~VM_STACK */
    unsigned long stack_vm;    /* VM_STACK */

       ...... 省略 ........
}
```
![[Pasted image 20240515161646.png]]

- 内核通过 `vm_area_struct` 的双向链表, 对用户空间的虚拟内存段进行管理, 每一段对应一个结构
![[Pasted image 20240515163614.png]]
```c
struct vm_area_struct {

	struct vm_area_struct *vm_next, *vm_prev;
	struct rb_node vm_rb;
    struct list_head anon_vma_chain; 
	struct mm_struct *vm_mm;	/* The address space we belong to. */
	
    unsigned long vm_start;     /* Our start address within vm_mm. */
    unsigned long vm_end;       /* The first byte after our end address
                       within vm_mm. */
    /*
     * Access permissions of this VMA.
     */
    pgprot_t vm_page_prot;
    unsigned long vm_flags; 

    struct anon_vma *anon_vma;  /* Serialized by page_table_lock */
    struct file * vm_file;      /* File we map to (can be NULL). */
    unsigned long vm_pgoff;     /* Offset (within vm_file) in PAGE_SIZE
                       units */ 
    void * vm_private_data;     /* was vm_pte (shared mem) */
    /* Function pointers to deal with this struct. */
    const struct vm_operations_struct *vm_ops;
}
```

- 虚拟内存区域可以映射到物理内存上，也可以映射到文件中，映射到物理内存上我们称之为匿名映射，映射到文件中我们称之为文件映射。
	- 如果申请的是比较大块的内存（超过 128 K）时，则会调用 mmap 在上图虚拟内存空间中的文件映射与匿名映射区创建出一块 VMA 内存区域（这里是匿名映射）。这块匿名映射区域就用 struct anon_vma 结构表示。
	- 当调用 mmap 进行文件映射时，vm_file 属性就用来关联被映射的文件。这样一来虚拟内存区域就与映射文件关联了起来。vm_pgoff 则表示映射进虚拟内存中的文件内容，在文件中的偏移。

![[Pasted image 20240515165744.png]]
- 内核态虚拟内存空间是共享的, 不同进程进入内核态之后看到的虚拟内存空间全是一样的
	- 直接映射区: 896MB, 直接映射到物理内存上, 存放的是内核的代码, 数据... + 每个用户进程的内核栈(固定大小)
	- 动态映射区: 128 MB, 与 3200 MB 剩余的高端物理内存进行动态映射, vmalloc 分配的内存在虚拟内存上是连续的，但是物理内存是不连续的。通过页表来建立物理内存与虚拟内存之间的映射关系，从而可以将不连续的物理内存映射到连续的虚拟内存上。
	- 永久映射区: 允许与高端内存建立长久映射关系
	- 固定映射区
	- 临时映射区


- 完整布局
![[Pasted image 20240515170223.png]]
![[Pasted image 20240515170325.png]]


- 物理内存: 随机访问存储器 `RAM` 
	- 静态 RAM, `SRAM`: 用于高速缓存
	- 动态 RAM, `DRAM`: 用于主存

- CPU MESI 缓存一致性协议
	1. **Modified（修改）**：当一个处理器修改了一个缓存行中的数据，并将数据写入缓存时，该缓存行被标记为 Modified 状态。此时，缓存行中的数据与主内存中的数据不一致。当其他处理器请求这个缓存行时，持有该缓存行的处理器需要将修改后的数据写回主内存，并将缓存行状态改为 Exclusive 或 Shared。    
	2. **Exclusive（独占）**：当一个处理器将一个缓存行从主内存中读取到自己的缓存中时，该缓存行被标记为 Exclusive 状态。此时，缓存行中的数据与主内存中的数据一致，且只有当前处理器拥有该缓存行。当其他处理器请求这个缓存行时，持有该缓存行的处理器需要将数据复制到请求的处理器缓存中，并将自己的缓存行状态改为 Shared。    
	3. **Shared（共享）**：当一个处理器将一个缓存行标记为 Shared 状态时，表示该缓存行中的数据与主内存中的数据一致，并且多个处理器都可以访问该缓存行。当其他处理器请求这个缓存行时，持有该缓存行的处理器需要将数据复制到请求的处理器缓存中。    
	4. **Invalid（无效）**：当一个缓存行不再有效时，它的状态被标记为 Invalid。这可能是因为数据已经从主内存中删除，或者是因为缓存行中的数据已经过时。


# 4.7 深入理解 Linux 物理内存管理
