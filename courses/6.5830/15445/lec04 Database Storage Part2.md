# Log-structure storage
上节课中的 tuple-oriented 通常有这些问题:
- 碎片: 通常无法填满整个 page
- 无效 IO: 为了从 disk 中拿到一个 tuple, 需要把整个 page 都放进 memory
- 随机 IO: 有可能会为了拿到 20 个 tuple 而把分别 20 个 page 都读进内存中

log-structure storage 通过规定只能对 tuple 写入(put)或者删除(delete), 而不允许对现有的 tuple 重写避免上述问题,
一般的状况是:
一个 log-structure storage page 在内存中, 可以不断地在这个 page 中写入 put 和 delete 的记录, 当这个 page 满了, 就将 page 放入硬盘中, 硬盘中的 log-structure storage 不允许重写, 只允许读

在 log-structure storage 中, 每一个 tuple 都有自己的 identifier, 用于唯一标识这个 tuple
![[Pasted image 20230910155341.png]]

优势: 使用上述 20 个 tuple 分布在 20 个 page 的例子,
log-structure storage 更新 20 个 tuple 只需要向已经在内存中的 page 写入 20 个更新记录(这是顺序写入, 不会有 disk io), 这样就避免了 tuple-oriented 中将 20 个 page 读入内存中, 让后再重写 tuple 内容

读数据:
维护一个 tuple index, 指向对这个 tuple 最近的一次更新
因此在获得某个 tuple 时, 只需要找到 tuple index 中对应的位置, 就可以读出对这个 tuple 的最近修改, 读出它的值

![[Pasted image 20230910160806.png]]

compaction:
这样子的操作是非常耗费空间的, 因此需要(定时)对数据库中的 pages 进行 compaction(也就是将对同一个 tuple 操作的记录归并成一个记录(也就是只保留最后一个记录))
![[Pasted image 20230910161144.png]]

在经过了 compaction 之后, 可以保证对于每一个 tuple, 只有一个记录在数据库中, 因此时间信息就不是非常重要了, 可以使用 tuple identifier 搜索, 此时为了方便搜索(), 可以将这些记录按照 tuple identifier 进行排序, 排序后称为 SSTable(sorted string table)

![[Pasted image 20230910162546.png]]
![[Pasted image 20230910162601.png]]

compaction 的方式:
![[Pasted image 20230910163807.png]]


劣势:
读写放大(对于一个只被 put 一次的 tuple (只有一个记录) 在 compaction 的过程中, 它会被不断地读写, 造成无效的开销)


# data representation
tuple 中只是一串字节顺序, 需要 DBMS 的解读才能读出它的意义
tuple 中通常可以存储 5 种高级数据类型
- Integers
多数 DBMS 都使用本机的 C/C++整型(固定字长)
例如: INTEGER, BIGINT, SMALLINT, TINYINT

- Variable-precision numbers
可变精度的, 同样使用本机 C/C++类型(固定字长)
例如: FLOAT, REAL

- Fixed-point precision numbers
具有任意精度的数字数据类型, 通常以精确的, 可变长度的二进制表示
此外还带有一些元数据(长度信息, 小数点位置信息)
例如: NUMERIC, DECIMAL

- Variable length values
可变长度数据, 通常是一些字符串, DBMS 通常不会允许一个 tuple 内的信息大小超过整个 page, 太大的数据可使用 overflow page 或 external file 存储
例如: VARCHAR, VARBINARY, TEXT, BLOB
![[Pasted image 20230910173558.png]]

- dates/times
时间数据
例如: TIME, DATE, TIMESTAMP

# system catalogs
存储关于数据库本身的元数据(数据库种有哪些表, 哪些列)