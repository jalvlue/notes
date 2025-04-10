# control: condition codes
- `registers` are processor states
- `%rax...`
- `%rsp`
- `%rip`
- `CF, ZF, SF, OF`, one-bit status reg, set by recent tests
	- `CF`, carry flag, for unsigned overflow
	- `SF`, sign flag, for signed(set when a negative number is produced)
	- `ZF`, zero flag
	- `OF`, overflow flag, for signed
- implicit setting: by logic and arithmetic operations
- explicit setting:
	- `cmp src2, src1`, like computing `a-b` without setting destination but setting status registers
	- `test src2, src1`, like computing `a&b` without setting destination but setting status registers
- `condition code` 有重要的意义, 不管是显式指令设置还是隐式指令设置, 后续都可以通过读取判断这四个 `CF, SF, ZF, OF` 来查看上次运算后产生了什么样的结果, 从而根据结果的情况来做分支跳跃

# reading condition codes
- `setX dest`, set low-order byte of destination to 0 or 1 based on combinations of condition codes, does not alter remaining 7 bytes
- ![[Pasted image 20250225144124.png]]
- since it would not alter remaining bytes, when it comes to setting cont, it is typically used with `movzbl` instruction to finish job (`movzbl` would set the upper 4 bytes to 0)
- ![[Pasted image 20250225144324.png]]

# conditional branch
- `jX`, jump to different part of code depending on condition codes
- ![[Pasted image 20250225144832.png]]
- `cmovX`, conditional move, for those eval both case, but set the result base on conditions
- ![[Pasted image 20250225145103.png]]

# loops
- `do-while`, `body - test - jump back to body if test ok`
- ![[Pasted image 20250225145221.png]]
- `while`
	- translation1: `jump to test - test - jump to body if test`
	- ![[Pasted image 20250225145509.png]]
	- translation2: `test - goto done if !test - body - test - goto body if test`
	- ![[Pasted image 20250225145613.png]]

# switch statement
- compilers use switch jump table to optimize sequential exec