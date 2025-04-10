GFS - the Google File System

# 0. Abstraction
GFS, 给大型数据密集的分布式应用提供存储服务的, 一个可拓展的分布式文件系统
分布式存储就是一大群便宜的机器一起存储着数据, 并且提供了 fault tolerance
并且为使用的用户(应该就是应用)提供了高聚合性能(==不懂==)


# 1. Introduction
GFS 继承了传统文件系统的功能: 性能, 可拓展性, 可靠性, 可用性

但有特别的设计:
1. 机器错误 (component failures)被视为常态 -->constant monitoring, error detection, fault tolerance, and automatic recovery
2. 文件格式、大小很重要, 数据通常是由小个的 web document object 组成的, 如果把每个 object 都单独存储为 KB-sized 文件, 系统的 I/O 开销非常大 --> design assumptions and parameters such as I/O operation and block sizes have to be revisited.
3. 多数文件进行修改的方式是新增一行, 而不是直接修改, 也就是已写入的文件是只读的 --> 新增成了性能优化和原子性保证的优化核心
4. 共同设计应用程序和文件系统提高了灵活性, 例如 GFS 提供了原子插入操作, 让不同的应用不加锁并发地插入

# 2. Design Overview
##### 2.1 assumptions
1. 系统由多个便宜机器组成, 因此需要定时检测错误, 容错, 从错误中恢复
2. 系统存储少量大文件, multi-GB 的文件是常见的, 需要进行优化, KB-sized 文件需要支持但是无优化
3. 两种 read workload: 大型顺序读取和小型随机读取 (large streaming reads and small random reads)
4. 两种 write workload: 大型顺序写入和小型随机写入 (无优化)
5. 并发写入数据到同一个文件时, 系统必须实现 `well-define` 的操作(处理并发) --> Atomicity with minimal synchronization overhead is essential
6. 高持续带宽比低延迟更重要, 大多数目标应用程序都重视以高速率批量处理数据，而很少有应用程序对单个读取或写入有严格的响应时间要求

##### 2.2 interface
GFS 提供了跟一般的文件系统(例如 posix 中定义的)类似的接口,
GFS 文件由层级结构组织, 使用路径定位

支持的常规操作: create, delete, open, close, read, write

支持的特殊操作:
1. snapshot: 以低成本创建文件或目录树的快照
2. record append: 允许并发地向同一个文件新增一行(append), 并保证 append 的原子性(用于实现多路合并结果和生产者、消费者队列)

##### 2.3 architecture
![[Pasted image 20231118120246.png]]
三种角色: 一个 master, 多个 chunkserver, 多个 client

文件都被分成固定大小的 `chunks`, 每一个 `chunk` 在创建时, 由 master 分配一个全局的不变的 64-bit `chunk handle`, 唯一标识

chunkserver 使用 linux file system 存储数据, 并按照指定的 `chunk handle` 和 `byte range` 读写数据

一个 `chunk` 默认被存储为 3 个副本, 为了可靠性

master 维持系统运转的所有信息, 就像图上的命名空间, 文件位置, chunk 位置, chunkserver 状态...

client 通过 API 与 master 交换元数据, 与 chunkserver 进行实际数据通信

##### 2.4 single master
单个 master 有许多好处: 不需要分享、同步一些 master,
但缺点是需要对 master 的读写进行优化, 不让单个 master 成为系统的 bottleneck

例子: 一个简单的读文件交互
client 将 app 要求的 filename, byte offset 转换成 filename 和 chunk index 发送给 master
master 查询信息并把 chunk handle 和 chunk location 返回给 client
client 拿着这些信息找到一个拥有数据副本的 chunkserver 并请求数据
chunkserver 收到请求就把数据返回

需要注意的是: master 和 chunkserver 都无缓存, 都会直接响应 client 的请求,
client 一般会缓存 app 的 filename 和 byte offset, 减少访问 master 的次数

##### 2.5 chunk size
GFS 选择了 64MB 作为 chunk size, 比多数文件系统都大得多
每一个 chunk 再 chunkserver 上作为普通的 linux file 存储

大 chunk size 的优点:
1. 多数 app 顺序读写一大片文件, 大 size 可以减少与 master 的交互,
2. 让 app 对数据的操作集中在少数几个或一个 chunk 中, 这样 client 就可以与 chunkserver 用一个 TCP 连接, 减少网络负担
3. 减少了 master 维护的元数据的大小, 让 master 数据可以放在内存中

缺点: 可能导致一个 chunk 被密集访问, 产生热点(hot spot)

##### 2.6 metadata
三类 metadata:
1. file and chunk namespace
2. mapping from files to chunks
3. locations of each chunk's replicas(mapping)
所有 metadata 都在 master 的内存中
值得注意的是 1、2 类 metadata(namespace, file-to-chunk mapping) 同时也是 N-V 的, 使用 log 机制写入 master 的硬盘中(并复制副本到另一台机器上)
log 机制更新更快, 更简单安全

第 3 类数据不是 N-V 的, 只有在 master 启动后, 新的 chunkserver 加入集群时, master 才会向它要 chunk location

###### 2.6.1 in-memory data structure
在内存中就是快, 做很多事情都方便了很多

但问题是内存可能不够, 但这不是问题, 对于一个 64MB chunk, metadata 也就是小于 64B 的大小, 就算真不够了加内存也很便宜方便

###### 2.6.2 chunk locations
就像上面说的, chunk location 只有在 chunkserver 加入时汇报给 master,
之后 master 通过与 chunkserver 之间的 heartbeat 获得最新的 chunkserver 信息

###### 2.6.3 operation log
log 包含了 metadata 的历史改变(因为 log 是以 append 形式追加上去的)
是 master 的持久存储, 被分散在 master 的磁盘和其他备份机器上

每次 master 会累积几条 log, 然后一起写入到磁盘上,
在没写入磁盘之前(内存磁盘未同步), master 是不会回复任何 client 请求的
保持一致性, 避免后续出现问题

master 出故障需要恢复时, 就需要使用 log, 重新执行 log 恢复文件状态
但 log 太大时恢复时间就长了, 而且做了很多重复工作(log 可能有很多对同一个 file/chunk 的操作, 但只有最后一个是有效的)
因此, 在 log 超过一定大小后, master 会建立一个 checkpoint,
此后恢复时, 只需要恢复到最近的 checkpoint 以及这之后的几个 log

checkpoint 是一种类似 B-tree 的紧凑结构, 对快速恢复很有效

##### 2.7 consistency model
GFS 有一个相对宽松的一致性模型, 因此简单但是不失高效

###### 2.7.1 guarantees by GFS
namespace 的改变是原子的(例如 file creation), 完全由 master 管理掌控


在数据发生改变后, file region 的状态:(取决于改变是否成功, 以及改变的种类和是否并发)
![[Pasted image 20231118164759.png]]
consistent: 所有 client 看到的数据都是一样的
defined: consistent, 并且所有 client 都明白写的是什么
undefined: 由于并发写入, 不同 client 可能交叉写入, 虽然结果对所有 client 都一样(consistent), 但没有 client 看得懂


###### 2.7.2 implications for application
使用 append 的方式更改文件, 而不是直接更改



# 3. System Interaction
原则是尽可能减少 master 的操作

##### 3.1 leases and mutation order
mutation 就是对 chunkserver 上的数据或 master 上的 metadata 进行修改(针对的是数据的所有副本)

metadata 完全由 master 管理, 因此只需要处理 master 及其远程机器上的副本的同步就好, 相对简单

chunk content 中, 则需要使用 `lease` 来保证各个 replicas 中数据一致性
![[Pasted image 20231121090202.png]]
lease 顾名思义就是一个合约的意思, 对于每一个 chunk, 只有一个 replica 拥有这个 chunk 的 lease, 因此称为 `primary`, 由 master 在生成 chunk 时指定, 使用 `primary` 来协调各个 replicas 的同步, 减轻 master 的压力, 

1. Client 每发出一个申请修改, master 会回复 primary 和其他 replicas 的位置, 没有 primary 就当即选一个
2. Client 把修改的内容/数据发给每个 replicas, chunkserver 会把这些东西存在 buffer 中
3. 所有 replicas 都回复收到数据后, client 向 primary 发出写入的请求
4. Primary 同时可能收到多个 client 的请求, 因此将这些请求排序并自己线性写入这些请求
5. Primary 写完把顺序同步给其他 replicas, 
6. Replicas 回复 primary 写完了
7. Primary 回复 client 修改了, 并把所有发生的错误 (replicas 错误, 或 primary 错误)发送给 client
8. Client 收到错误, 就表示修改属于 inconsistent, 被认为是失败了, 虽然在部分 replicas 中写入
9. Client 自己的代码决定是重新修改还是放弃


##### 3.2 data flow
实际上上面第 2 步 client 将所有修改数据发给所有每个 replicas 并不是直接由 client 发送给每个 replicas,
为了利用每一台机器的出口带宽, 
client 会将数据先发给距离他最近的 chunksserver S 1
S 1 再转发给离他最近的 S2...
直到覆盖所有 replicas,
因此数据流动是线性的

##### 3.3 atomic record append
GFS 对于 record append 是支持并发, 而且原子的
因此 record append 是更好的更支持的一种修改方式

record append 与直接修改有一些类似的地方,
都由 `primary` 协调, 但在这里, 一旦有 replicas 出现失败,
`primary` 会让所有 replicas 都重传, 因此可能会导致重复,
因此是 `at lease once` 的

##### 3.4 snapshot



# 4. Master Operation


# 5. Fault Tolerance and Diagnosis


# 6. Measurement


# 7. Experience


# 8. Related Work


# 9. Conclusions
