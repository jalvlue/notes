[Zero-copy: Principle and Implementation](https://medium.com/@kaixin667689/zero-copy-principle-and-implementation-9a5220a62ffd)
[mmap() vs. reading blocks](https://stackoverflow.com/questions/45972/mmap-vs-reading-blocks)
[When should I use mmap for file access?](https://stackoverflow.com/questions/258091/when-should-i-use-mmap-for-file-access)

# Overview
- `mmap` 是一个 `posix-compliant Unix syscall`
- 主要的作用是将文件内容直接映射到用户可访问的内存中, 减少 `read, write` 中数据从内核缓冲区的多一次流转
- 由于是将文件内容映射到用户可访问的内存中, `mmap` 也有申请内存的功能, 因此对于大内存分配需求, 直接使用 `mmap` 映射一个匿名文件来获得一大块可用内存是非常高效方便的
- 同时 `mmap` 还可以让多个进程共享同一个内存页





# IO background
- `disk IO`: bolck-oriented, 阻塞式的
- `network IO`: stream-oriented, 流式的
- 传统的 `read, write syscall`
![[Pasted image 20240816105937.png]]
- 数据会在 `DMA --> kernel buffer --> user space` 中进行流转
- 用户无法直接操作操作内核中的数据, 同时为了避免恶意文件对用户进程空间的破坏, 这种流转保护是必须的
- 虽然 DMA 已经释放了一部分 CPU 压力了, 但是 `kernel buffer --> user space` 之间的拷贝还是给 CPU 带来了性能损失


- 再加上通过 `kernel socket interface` 将数据发送给客户端的流程后, 性能浪费更加明显![[Pasted image 20240816111613.png]]
- 这里 CPU 根本不需要做任何的计算工作, 但是经历了 2 次数据拷贝, 4 次用户-内核态转变


# Zero-Copy
- `zero-copy` 零拷贝, 是一种通用的概念, 主要是减少 CPU 的拷贝, 以提高性能有许多达到零拷贝的具体实现, `mmap` 就是一种
- `mmap, Memory Mapping`
- 操作系统的虚拟内存允许将多个虚拟内存页映射到同一个物理内存页, `mmap` 就是利用了这个性质直接将某个用户虚拟内存页映射到跟内核缓存虚拟页相同的物理内存页上, 来减少一次 `kernel buffer --> user space` 的拷贝, 也就是, 用户页和内核页共享同一个实际物理页
- ![[Pasted image 20240816112719.png]]
- 使用了 `mmap` 之后, CPU 数据拷贝减少了一次, 而且只发生在内核中, 数据不会流转到用户空间

