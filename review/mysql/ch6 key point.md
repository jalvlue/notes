- 日志
	- undo log(回滚日志), 实现了事务的原子性, 主要用于回滚的MVCC
		- ![[Pasted image 20240521160241.png]]
		- 通过 `roll_pointer` 指针记录 undolog 的版本链
	- redo log(重做日志), 实现了日志的持久性, 主要用于故障恢复
	- binlog(归档日志), 主要用于数据备份和主从复制
	- 