# storage

volatile devices:
易失性存储设备(断电数据就没了)
包括 register, cache, main memory
统称为 memory

non-volatile devices:
非易失存储设备(断电数据还在)
包括磁盘(SSD, HDD)
统称为 disk

课程主要专注于"disk-oriented" DBMS, 也就是数据库主要存储于磁盘
因此数据在 memory 和 disk 之间流动的过程中, DBMS 需要承担责任(因为需要将数据调进内存才能进行处理)
但是将数据从磁盘读进内存相当耗时, 此外磁盘数据是按页读写的, 一次会读一整页, 即使你只需要一小部分, 因此为了减少往复读写带来的时间开销, DBMS 应该将顺序读写(seqential access)最大化, 并尽量避免随机读写(random access)

总结:
数据库位于磁盘中, 按分页的方式组织(第一页是目录页)

DBMS 通过拥有一个 buffer pool 管理数据页在磁盘到内存之间的流动
DBMS 还拥有一个 execution engine, 用于执行查询, 执行查询过程中, engine 会向 buffer pool 指定一个页(可能多次), 然后 buffer pool 将这一页数据放进内存中(os 的细节先忽略了), 并给 engine 返回一个指向这一页的指针, 并保证在 engine 使用这一页时, 这一页都会在内存中


# file storage
几乎所有 DBMS 都有自己的文件存储格式, 因此只有自己可以解码这些文件
由上, 数据库按分页的方式组织, 因此 DBMS 的存储管理(storage manager)需要管理这些页(追踪哪些数据被读/写进哪一页, 哪一页还有多大的空余空间)


# database pages
数据库被存放在一个或多个文件中, 
文件按分页方式组织, 页面大小通常是固定的
数据库中会存在多种数据(tuples, indexes,...), 一般来说, 一个页面中只会存放一种数据, 
页面通常是 self-contained 的, 也就是所有 storage manager 对页面想知道的信息, 都是存放在这个页面里的
每个页面都会有一个自己的标识符(unique identifier), 唯一识别这个页面

![[Pasted image 20230909235117.png]]
硬件可以保证对于 4KB 的页面的写入是原子的,
但如果数据库页面大小大于 4KB, DBMS 就需要使用一些措施保证对页面写入的原子性


# database heap file
heap file 是 pages 的无序集合, 使用 heap file 是一种定位 page 的方式,

通常上层应用向 heap 发出需要 page 的信息(页号之类的), heap 再返回这个页
因此需要使用链表或目录(映射)来组织
![[Pasted image 20230910004253.png]]

# page layout
页面通常分为两部分: 
header: 关于这一页的元数据(页面大小, 校验和, DBMS 版本,...)
storage: 实际存储的数据(分为两类: tuple-oriented, log-structure)

slotted pages: tuple-oriented
![[Pasted image 20230910004541.png]]
使用 slot 映射 tuple, 增长顺序相反, 因此在容量满了的时候可以知道
删除或增加 tuple 只需增加映射
通常会定时做 compact 工作


# tuple layout
header: 关于这个 tuple 的元数据
data: 实际的数据
Unique identifier: 也就是对应的 slot
