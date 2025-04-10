TODO: 太难看了, 看不懂, 有空学懂了再回来看把


# Introduction
- `data race` 发生在多个 `goroutines` 同时读写同一个内存地址时
- `DRF-SC, data-race-free programs execute in a sequentially consist manner`, 程序没有 `data race` 时, 可以认为所有 `goroutines` 都复用了一个 CPU, 此时就是 `DRF-SC`


# Memory Model
- 内存模型描述了程序执行的需求
	- 程序执行需求由 `goroutine` 执行组成
	- `goroutine` 执行由内存操作组成
- 一个内存操作 `memory operation` 需要从四个方面描述
	- `kind`, 操作类型
		- 普通读写
		- 同步操作: `atomic, mutex, channel`
	- `location`: 发起操作的位置
	- `access location`: 操作的变量内存地址
	- `values`: 操作的值
- 一个 `goroutine` 的执行在内存模型中, 就是一组内存操作
- 