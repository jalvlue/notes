- 其他杂项

# Process Memory Layout
- `x86-64 linux process`
	- `stack`, runtime stack for procedure call and local variable
	- `heap`, for dynamic managed memory
	- `data`, for global variable and static variables
	- `text`, code instructions
- ![[Pasted image 20250317203544.png]]
- ![[Pasted image 20250317203600.png]]
- ![[Pasted image 20250317203615.png]]

# Buffer Overflow
- 进程栈是连续的内存空间, 需要给数据预先分配空间, 同时多个数据在内存上是连续的, 而且 C 语言支持通过指针直接操纵内存, 因此恶意的写入过多数据/覆盖其他数据这种事情非常容易出现 `code injection, buffer overflow`
- 同时进程栈是空间有限的, `x86-64 Linux` 预分配 `8 MB` 栈空间, 太深的递归函数调用或太大的局部变量可能会导致栈溢出 `stack overflow`
- ![[Pasted image 20250317204004.png]]
- ![[Pasted image 20250317204021.png]]
- ![[Pasted image 20250317204033.png]]

# Unions
- 联合体, 类似结构体, 但是多个变量共享同一块内存
- 底层位模式是同一个, 只是根据使用的类型, 通过编译器来使用不同数据类型的指令
- ![[Pasted image 20250317204350.png]]
- ![[Pasted image 20250317204408.png]]

# Summary
![[Pasted image 20250317204424.png]]
