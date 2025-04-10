# fractional binary numbers
- 二进制分数, 这种表示方式和现实世界十进制小数非常类似, 只是采用了 `base 2`
- 由于是 `base 2`, 因此有一些实数是无法使用有限的二进制分数表示出来的
- 只使用二进制小数点来区分整数和小数部分, 确定这两部分的大小非常麻烦
![[Pasted image 20241215223228.png]]

# IEEE floating point
- `IEEE` 浮点数标准
- 所有的实数都可以近似用这个公式表示出来 `(-1)**s * M * 2**E`
![[Pasted image 20241215224411.png]]
- 在 `IEEE floating point format` 中
	- `s`, 一位符号位
	- `exp`, 数位指数位, 编码了公式中的 `E`
	- `frac`, 数位尾数位, 编码了公式中的 `M`

![[Pasted image 20241215224603.png]]
- 不同的精度下不同格式的长度
	- `single-precision, 4-bytes`, 也就是 c 中的 `float`
	- `double-precision, 8-bytes`, 也就是 c 中的 `double`


# normalized value
- 规格化的浮点数, 这里的想法是将小数规格化 (表示为 `(-1)**s * M * 2**E`)
	- `exp != 000...0 && exp != 111...1`, 这两个后续有涉及到, 如果为这两种, 则不属于规格化的解释范围
	- `s`, 符号位, 正数为 0, 负数为 1
	- `E`, 指数位, 使用 `exp` 编码表示, `E = exp - bias, bias = 2**(k-1) - 1`
		- 对于 `single-precision`, `k = 8`, 也就是说, `bias = 127`
		- 对于 `double-precision`, `k = 11, bias = 1023`
	- `M`, 小数位, 使用 `frac` 编码表示, 规格化的编码总是把小数部分移动成 `1.xxx * 2**E`, 因此总会有一个 1 在最前面, 从而就把这个 1 省略了

![[Pasted image 20241216000108.png]]
![[Pasted image 20241216000119.png]]
- 总结: 使用 `exp` 编码 `E` 的原因是, `exp` 可以看成是一个 `k-bits unsigned`, 两个浮点数比较可以直接比较 `exp`, 非常直观

# de-normalized value
- 规格化的浮点数表示很好, 但是不够好, 由于小数部分一定有一个前导 1, 因此无法表示 0, 而且不能很好地表示接近 0 的数
- 因此引入非规格化的浮点数格式, 表示 0 和靠近 0 的数
	- `s`, 符号位
	- `exp = 000...0`, 非规格化的指数位全为 0, 此时 `E = 1-bias, single-precision: -126, double-precision: -1022`, 区别于非规格化的 `E = exp-bias`
	- `frac, M=0.xxx`, 非规格化的小数位没有前导 1, 是大多就是多大, 因此可以表示非常接近 0 的小数

![[Pasted image 20241216001516.png]]
- 引入 `denorm` 之后, 不仅可以表示非常接近 0 的数, 而且还可以用全 0 表示 0, 非常直观

# special values
- `exp = 111...1`, 全 1
	- `frac = 000...0`, 表示无穷, 正无穷或负无穷
	- `frac != 0`, 表示非数 `NaN`
![[Pasted image 20241216001814.png]]


# summary
- 整个浮点数表示数轴
![[Pasted image 20241216001852.png]]


# example: 8-bits float
- `s, 1-bit`
- `exp, 4-bits`
- `frac, 3-bits`

![[Pasted image 20241216002016.png]]
- 可以看出, `norm-denorm` 之间的数字大小转移非常丝滑
![[Pasted image 20241216002207.png]]
![[Pasted image 20241216002217.png]]


# rounding
- 浮点数运算结果可能会超出表示范围, 此时就必须丢失一些精度, 从而尽可能地保存信息 (区别于整数溢出直接就是数学上取模回到结果区间, 浮点数的运算溢出只会到达负无穷和正无穷, 其他情况均为精度丢失)
- 因此怎么保存精度非常重要


![[Pasted image 20241216002645.png]]

- 四种常见的 `rounding strategy`
![[Pasted image 20241216002729.png]]


# round-to-even
- 这是 IEEE 默认的浮点数 `rounding strategy`
![[Pasted image 20241216002828.png]]
![[Pasted image 20241216002834.png]]


# multiplication
