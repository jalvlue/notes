# database workloads

OLTP: On-Line Transaction Processing
OLTP 的特点是运行快, 运行时间短, 查询操作简单(但是通常重复), 通常 write 比 read 多
例子: Amazon storefront, 用户将商品加入他们自己的购物车(write), 付款更新购物信息(写), 这些行为只会影响他们的账户, 但是可能会有很多账户进行这样的操作(简单但重复)
![[Pasted image 20230912105049.png]]
(做小改动(update, insert))


OLAP: On-Line Analytical Processing
OLAP 的特点是运行时间久, 查询操作通常很复杂, 要从数据库中查询大量数据(read), 通常用来根据现有的数据分析推断出一些有用的新的商业分析
例子: Amazon 可能会查询数据库中 Pittsburgh 最畅销的商品(获得新的信息), 然后基于这些数据做出某些商业决策
![[Pasted image 20230912105123.png]]
(做大查询)

HTAP: Hybrid Transaction + Analytical Processing
结合了 OLTP 和 OLAP, 同时在同个数据库中做这两个操作

![[Pasted image 20230912104420.png]]


# storage models

N-Ary Storage Model (NSM)(row store)
n 元模型就像 lec2,3 中说的一样, 默认一个 tuple 中的 n 元都是存储在同一个 page 上的, 因此对于 OLTP 这种写密集型的, 一次只修改一个实体(tuple)的操作, n 元模型非常理想
![[Pasted image 20230912110857.png]]

但不适合大幅度的扫描一个表的大部分(tuple)或者一些属性(表的子集)(会将很多的不必要数据也带进内存中(在同一个 page 必须一同拿进内存))
![[Pasted image 20230912110924.png]]

Decomposition Storage Model (DSM)(column store)
DSM 将一个 tuple 中的属性都拆开, 并将同类的属性值存在同一个 tuple 中, 再将同类的 tuple 存在同一个 page 中
![[Pasted image 20230912111924.png]]
此时再进行 OLAP 这样的大幅扫描整表的操作就不会将 useless data 带进内存, 降低了 IO 浪费
此外, 由于同类属性的值通常在统一范围内, 因此数据更紧凑

但是在进行单个查询, 插入, 修改时(OLTP), 由于属性数据是分散的, 因此会增加额外的开销

将分散属性数据组织的方法:
1. Fixed-length offset (preferred)
按顺序存储, 属于同一组记录的属性数据在他们各自存储的 tuple 中都有相同的 offset, 因此可以根据这个相同的 offset 在各个属性 tuple 中找到并组织同一记录的属性数据
![[Pasted image 20230912113019.png]]
例如 ABCD 是记录的 4 个属性, 因此他们被存储在 4 个 tuple 中, 对于同一记录的数据, 在各自的 tuple 中有相同的offset

2. Embedded Tuple Ids (less common)
对于一个属性数据, DBMS 存储一个 id, 并存储到同一记录别的属性数据的隐射, 开销巨大(很少使用)
![[Pasted image 20230912113442.png]]

总结: 如果想要一个高性能的系统(OLTP, OLAT 都考虑到), DSM(column store)会是一个更好的选择


# database compression : speed vs. compression ratio
1. 现实世界中的数据通常都是 well-defined & highly skewed (高度歪曲的, 例如有一个东西多次出现, 而别的东西出现次数远远少), 
2. 现实世界的数据中, 同一个记录的属性通常会有很高的相关性
因此根据这些数据的特性, 可以对数据进行压缩

compression Goals:
1. 必须生成定长的值 (便于使用 offset 访问), 变长的数据是一个例外, 可以使用指针之类的来处理
2. 执行查询时, 尽可能让查询在压缩后的数据库进行 (late materialization)
3. 压缩必须是无损的

compression granularity:
1. block-level
2. tuple level
3. attribute level
4. columnar leve

# naive compression
朴素压缩, 按粒度属于 block-level, 只是简单的将一页压缩得更小, 每次在使用时(例如读写时), 需要讲压缩的页放进 buffer pool 中进行解压缩, 解压之后才能使用, 开销大,
例如: MySQL 将 16KB 的页压缩成[1,2,4,8]KB 大小, 每次要使用时, 放进 buffer pool 中解压
![[Pasted image 20230912135913.png]]
此外, 朴素压缩只是压缩, 不能知道压缩后的文件内容是什么(只能通过解压的方式知道), 因此丢失了 late materialization 性质, 局限很大

# columnar compression
1. Run-Length Encoding (RLE)
将连续的相同的属性值存在一个三元组(value, start position, elem_nums)中
![[Pasted image 20230912140619.png]]
直接压缩的效率可能不高, 可以引进排序, 使得相同值连续性提高
![[Pasted image 20230912140857.png]]

2. Bit-Packing Encoding
当属性的值总是小于类型的最大值时, 将它存储为较小的类型
![[Pasted image 20230912141320.png]]

3. Mostly Encoding
类似 bit-packing, 当出现离群值时, 对离群值使用特殊的记号标记, 别的数据使用较小的类型存储
![[Pasted image 20230912141802.png]]

4. Bitmap Encoding
使用 one-hot 编码
![[Pasted image 20230912142525.png]]
(有时候属性的值种类太多时会很大, 不好)

5. Delta Encoding
对于连续的属性值, 可以存储增量
![[Pasted image 20230912142714.png]]

6. Incremental Encoding
对于 string 的增量存储(根据 prefix 或 postfix)
![[Pasted image 20230912142938.png]]

7. Dictionary compression
将常用的, 变长的模式(例如字符串)替换成定长的, 更小的索引码, 再建立一个映射, 使得可以通过索引码得到原来的值
![[Pasted image 20230912143753.png]]

为了支持某些比较(例如大小比较), 原数据和映射数据可以相互转换,
让原数据和映射后的数据都表现出相同的比较性质, 可以对映射值排序后再赋索引码
![[Pasted image 20230912145129.png]]