# Bit pattern
- 计算机选择了二进制
- 因此使用二进制位模式来表示数据
- 最典型的, 数字用二进制表示:
	- 正整数直接使用二进制
	- 小数使用一个二进制点将正权和负权隔开
![[Pasted image 20241213121249.png]]
- 思考: 因此正整数的准确的, 而小数部分只能使用 2 的负权来逼近, 是不准确的

# Encoding byte values
- 二进制位机器看得懂, 但是太长了人看不懂
- 因此引入 `byte = 8 bits`
- `2 ** 8 = 256`, 因此一个字节最多可以表示 `256` 个位模式, `0~255, 0x0~0xFF`
- 十六进制表示法是更进一步的表示方式, 将 `4 bits`, `2 ** 4 = 16` 种位模式单独拿出来做展示, 一个 `byte` 可以使用两个十六进制数字表示出来
	- `1111-F`
	- `1010-A`
	- 只需要记住这两个, 其他的 `BCD` 在中间插值即可


# Typical C data representation
![[Pasted image 20241213122045.png]]


# Boolean algebra
- 逻辑代数, 对位模式上的操作非常重要, 也可以看作是 cpu 提供的接口
	- `and, &`
	- `or |`
	- `not ~`
	- `xor ^`
	- 特别地, 这些都是 C 语言的操作符

![[Pasted image 20241213122430.png]]


# General Boolean algebra
- 单独的真假逻辑代数作用很大但是还不够大
- 二进制位模式天然就是一个 `bit vector`
- 拓展的逻辑代数就是在整个 `bit vector, bit pattern` 上对应的每一#位都做代数
![[Pasted image 20241213122814.png]]

# Example: representing and manipulating set
- 位模式可以作为集合, 每一位表示对应元素的有无
- 位运算可以在常数时间对集合做操作
![[Pasted image 20241213123139.png]]

# Contrast: logic operations in C
- 一些 C 语言的比较运算符习俗, 区别于位操作
	- 做非永远都只会得到 `0, 1`
	- 比较逻辑提前结束
![[Pasted image 20241213123352.png]]


# Shift operations
- 位模式左移右移
- 逻辑位移: 左移高位补 0
- 算术位移: 左移高位补最高位, 这里的特性是对于 `int`, 算术左移在丢失精度之前不会改变数字的大小 (这点很有用!)
- 未定义行为: 如果移位负数位或者移位超过该数据类型位宽则会发生未定义行为


# Encoding integers, unsigned and signed
- 用二进制模式表示整数
- `unsigned`, 无符号整数
	- `B2U, binary to unsigned`, 直接把所有为 `1` 的位权重加起来
	- ![[Pasted image 20241214203646.png]]
- `two's complement`, 补码, 通常有符号整数都是这样表示的
	- `B2T, binary to two's complement`, 最高位的权重位负权, 也是直接把所有为 `1` 的位权重加起来
	- 由于最高位的负权非常重, 因此最高位为 `1` 则为负数, 否则为非负数
	- ![[Pasted image 20241214203854.png]]


# Numeric ranges
- `w` 表示二进制模式的位数
- `unsigned`
	- `UMin = u0, b000...0`
	- `UMax = u(2**w - 1), b111...1`
- `two's complement`
	- `TMin = t(-2**(w-1)), b100...0`
	- `TMax = t(2**(w-1) - 1), b011...1`
	- `TMinusOne = b111...1`
	- `TZero = b000...0`
- 总结, 对于一个长度为 `w` 的二进制位模式集合, 一共有 `2**w` 种组合, 也就是说, 不管是使用 `unsigned` 还是 `two's complement` 去解释, 都会只有这么多的解释方式, 因此他们能表示的范围大小都是相同的
- `two's complement` 可以看成是将一半的范围用来表示负数了 (最高位决定一半), 这是为什么会有很多 `w-1` 的原因
![[Pasted image 20241214205327.png]]

- 还有一个重要的推论, 对于位模式相同的 `unsigned, two's complement`, 他们总是相等 (非负数), 或者相差 `2**w` (负数), 这很直观, 因为最高位给 `unsigned` 带来了 `+2**(w-1)`, 给 `two's complement` 带来了 `-2**(w-1)`, 因此相差 `2**w`
![[Pasted image 20241214212122.png]]

- `c programming`, 与这些类型最大最小值相关的常量都在标准库 `limits.h` 里
```c
#include <stdio.h>
#include <limits.h>

int main() {
  printf("%d\n", INT_MAX);
  printf("%u\n", UINT_MAX);
  printf("%lu\n", ULONG_MAX);
  printf("%ld\n", LONG_MAX);
  printf("%ld\n", LONG_MIN);
}

```

# Casting surprises
- `unsigned <==> two's complement`, 相同位模式相互转换, 使用位模式做中介
- `U2T, unsigned to two's complement`
- `T2U, two's complement to unsigned`
![[Pasted image 20241214212303.png]]

- `c unsigned <==> signed` 之间的转换通常会维持位模式相同
- 涉及到比较的隐式转换时, `unsigned` 优先级更高
	- 如果比较的操作数中有一个是 `unsigned`, 那么所有操作数都会被隐式转换为 `unsigned` 然后再进行比较
	- 比较坑


# Sign extension
- 将一个 `signed int` 拓展的方法是在前面补符号位
	- 非负数补 `0` 无变化
	- 负数补 `1` 会导致权值与原先的相互抵消最终还是得到原来的符号权值

![[Pasted image 20241214223619.png]]


# Truncating
- 将一个位模式缩短, 直接裁掉前面的部分
- 从位模式的角度上看就是对需要部分进行取模

# unsigned addition
- 计算机的内存有限, 因此给每种 `unsigned` 类型都限定了固定的字节, 因此他们无法表示超出范围的数字
- 加法是可能超出表示范围的, 超出后计算机的做法是直接将超出的进位丢弃
- 两个 `unsigned` 相加, 范围只可能在 `0~2**(w+1)`, 因此只可能会有一个溢出位, 将这个位丢弃的效果其实在数学上就类似于将溢出结果对 `2**w` 取模
![[Pasted image 20241215170406.png]]

- 数学上应该出现的结果:
![[Pasted image 20241215170610.png]]

- 计算机中受限的结果: 悬崖效应
![[Pasted image 20241215170634.png]]


# two's complement addition
- 常见的 `signed int` 加法, 从二进制模式行为看来, 跟 `unsigned` 一摸一样, 只是解释方法不同, 因此在溢出时造成了更大的悬崖效应

- `TAdd and UAdd have identical bit-level behavior!!`
![[Pasted image 20241215170955.png]]

![[Pasted image 20241215171118.png]]
![[Pasted image 20241215171159.png]]

# multiplication
- `signed/unsigned int` 乘法, 这里不多涉及二进制乘法的方式
- 与加法类似, 溢出时都需要舍弃至原先的 `w` 位
- 但是数学上 `w` 位乘法结果最大可能涉及到 ` 2*w` 位, 从而舍弃高 ` w ` 位, 区别于加法只可能溢出一位
![[Pasted image 20241215172329.png]]
![[Pasted image 20241215172404.png]]
![[Pasted image 20241215172412.png]]


# power of 2 multiplication with shift
- 位模式就是基于二进制的, 因此对 2 进行乘除可以方便地使用位移操作, 编译器也会做一些优化
- 历史原因, 机器指令中, 乘法指令往往需要更多的时钟周期来执行, 使用位移操作效率更高
- 用移位来做高效的除法运算也是算数右移出现的主要原因
![[Pasted image 20241215173236.png]]
![[Pasted image 20241215173246.png]]

# byte-oriented memory organization
- 从字节的角度上看, 内存就是一个巨大的字节数组, 而且其中的每一个字节都有字节的编号 (地址)
- 程序就是通过地址来访问对应的字节中存的数据
![[Pasted image 20241215180157.png]]

# work-oriented memory organization
- `word`, 字长, 通常是该机器 CPU 在算数运算中, 通常处理的数据大小, 例如 32 位 CPU 的 `word` 就是 `32-bits, 4-bytes`, 表示通常处理 `4-bytes` 数据, 同时内存的最大寻址位也是 `32-bits`
- 64 位 CPU 则为 `64-bits, 8-bytes`
- 从字长的角度上看, 将字节聚合成字长, 并一次处理字长大小的数据, 这样对 CPU 的 cache 友好 (引入 `word padding`)
![[Pasted image 20241215180617.png]]
![[Pasted image 20241215180642.png]]


# byte ordering
- 字节序 [[little-endian 和 big-endian]]
- 字节序实际上就是一个 `word` 的数据内部的顺序
![[Pasted image 20241215180808.png]]


# Representing integer
```c
typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
  size_t i;
  for (i = 0; i < len; i++)
    printf("%p\t0x%.2x\n", &start[i], start[i]);
  printf("\n");
}

```

- `int a = 15213;` 的字长表示为 `0x 00 00 3B 6D`
- 按照 `intel little endian`, 低字节在高地址
![[Pasted image 20241215180843.png]]
![[Pasted image 20241215181109.png]]
![[Pasted image 20241215181101.png]]


# representing pointers
- 指针也是变量, 在字长内也是使用 `little endian representation`
![[Pasted image 20241215181428.png]]
- 可以注意到 `x64` 的高字节中, 有 `0x 00 00 7f`, 这是因为目前的 `x64` 计算机只是用了低 `47` 做寻址, 同样也是 `little endian`


# representing string
- 字符数组就很简单了, 从开始一直往下排, 没有字长内顺序的说法
![[Pasted image 20241215181713.png]]
