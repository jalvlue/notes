- 与读 SQL 比起来, `update` 这种写的 SQL 还需要涉及到日志模块
	- `redo log`: 重做日志
	- `binlog`: 归档日志

# redo-log
- 物理日志
- 是 `innodb` 特有的日志
- `redo-log` 实际上就是 mysql 实现 `WAL-write ahead logging` 的方式, 通过先写 `redo-log` 以及内存中 `buffer pool` 中的数据, 后续空闲了再同步到磁盘的方式来提高效率
- `redo-log` 是循环 `append-only` 的所以比随机写磁盘更快
- `innodb` 的配置中 `redo-log` 的大小是固定的, 一般是 `4*1GB`, 如果 `redo-log` 写满了, 但是系统还未空闲, 那么会先放下手中的活, 先把日志同步到磁盘中 (性能问题可能出现的地方)
![[Pasted image 20240903093310.png]]

- 有了 `redo-log` 之后, mysql 就是 `crash-safe` 的, 数据库异常重启之后也能恢复正常


# binlog
- 逻辑日志
- `binlog` 是 server 层自己的日志, 所有的底层引擎都可以使用

- 与 `redo-log` 的区别
	- `redo-log` 是 `innodb` 引擎独有的, `binlog` 是 server 层实现的, 所有的引擎都可以使用
	- `redol-log` 是物理日志, 记录了 "在某个数据页进行了什么修改", `binlog` 是逻辑日志, 记录了 SQL 的原始逻辑, "给 ID=2 这一行的 c 字段加 1"
	- `redo-log` 是循环写的, 空间会用完, 用完需要将之前的一部分日志同步到磁盘中, `binlog` 不会覆盖, 写完文件到一定大小就换文件


# update
0. `UPDATE t SET c=1 WHERE id=1`
1. 执行器先调用 `innodb` 的接口, 把 `id=1` 的记录拿出来 (主键搜索, 直接在聚簇索引上), 如果该数据页已经在内存中则直接拿, 不在则先把整个数据页从磁盘中加载到内存中, 然后再取出返回该记录
2. 执行器修改记录 `c=1`, 生成一条新的记录, 调用 `innodb` 的写接口将记录写回
3. `innodb` 首先更新内存中的数据页, 然后将操作记录到 `redo-log` 中, 此时 `redo-log` 完成执行器的任务, 状态改为 `prepare`, 随时等待提交事务
4. 执行器生成操作的 `binlog` 记录, 并写入
5. 执行器调用 `innodb` 的提交事务接口, `innodb` 将 `redo-log` 的状态改为 `commit`, 更新完成
![[Pasted image 20240908165926.png]]


# 两阶段提交
- 上面对于 `redo-log` 的操作分为了 `prepare-commit` 两个阶段的提交
- 这样做是保证了两份日志的逻辑一致, 如果 `redo-log` 是一次提交, 那么先写 `redo-log` 或先写 `binlog` 时, 如果数据库崩溃时一个写了一个没写都会导致问题
	- 一般都是在恢复全量备份之后, 从该备份的节点开始使用 `binlog` 恢复
	- `redo-log -> binlog`: 使用 `binlog` 恢复, 但是没有写 `binlog`, 数据丢失了
	- `binlog -> redo-log`: 使用 `binlog` 恢复, 但是这个 `redo-log` 是未提交的无效事务, 多出一个事务来, 与原库不同
- 因此 `redo-log` 的两阶段提交是很有必要的


# Question
```ad-question
:> 前面我说到定期全量备份的周期“取决于系统重要性，有的是一天一备，有的是一周一备”。那么在什么场景下，一天一备会比一周一备更有优势呢？或者说，它影响了这个数据库系统的哪个指标？

:> 最长回复时间比较短, 因为恢复也需要时间, 影响 RTO, recovery time object, RPO, recovery point object
```
