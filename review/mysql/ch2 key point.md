# 执行 select 语句

# 一行记录的存储
- 在 `/var/lib/mysql/dbname/tablename.idb` 中存储一个表的数据
	- 表
	- 段: 表的组成单位, 分为
		- 数据段
		- 索引段
		- 回滚段
	- 区: 数据量大时, 为了避免相邻的页物理距离太远, 造成太多磁盘随机 IO, 此时通过区分配(一个区为 `1MB`, 可将 64 个连续的页放在一起)
	- 页(InnoDB 按页将数据读入内存中, 默认为 `16KB` 一页)
	- 行(表中每一行数据)
- 行格式 `compact`
	- 额外信息
		- 变长字段长度 (`varchar, text, blob`), 逆序 (为了 CPU chche hit), 可有可无(全部列均为定长)
		- NULL 值列表 (字节对齐, 用一个 `bit` 标识某一个列是否为), 逆序 (为了 CPU cache hit), 可有可无 (全部列设置为 `not null`)
		- 记录头信息
			- `delete_mask`
			- `next_record`
			- `record_type`
	- 真实数据
		- `row_id`
		- `trx_id`
		- `roll_pointer`
		- n 个列的真实数据
- 行溢出: 列太大, 一页存不了一个行, 溢出到 `溢出页` 中
![[Pasted image 20240520170351.png]]
![[Pasted image 20240520164730.png]]


