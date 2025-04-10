# introduction
disk-oriented database 中, DBMS 必须将数据读进内存之后才能对数据进行操作, 
控制数据在 memory 和 disk 之间流通这一操作的 DBMS 组件称为 Buffer Pools, 

为了数据库的高效, Buffer Pools 通常需要保证两种控制
1. spatial control: 将 pages 写入到 disk 的什么位置(让经常同时使用 pages 在 disk 上物理距离足够近)
2. temporal control: 什么时候将 pages 读进 memory, 什么时候将 pages 写回 disk (为了最小化磁盘 io )

![[Pasted image 20230919111501.png]]


# locks vs. latches



# Buffer Pool
buffer pool 实际上是内存的一块, 由 DBMS 拥有, 里面存储了 disk 中的 pages 的副本, 因此也可以看作是 disk 的 cache, 

buffer pool 按 DBMS 的 page 大小分成一块一块固定大小的 frame (类似 os )(DBMS 有多种 page 大小时, 将会用多种大小的 buffer pool 处理对应大小的 pages)
