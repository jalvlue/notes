- [x] Lecture video
- [x] Lecture note

[[12 Hashing]]


# using data as an index
- 利用数组随机存取的特性
- 哈希的想法就是将数据本身映射成数组索引的 (哈希函数)
- 从而达到 `O(1)` 的写入, 读取


# hash code
- 考虑到整形可能会溢出, 而且通常情况下, 暴力方法获得的索引值都可能远大于溢出上界, 因此使用有固定范围的, 可以使用整形表示的 `hash code`
- 由于原数据集可能无限大, 结果集 (hash code) 有限大, 根据鸽巢原理, 一定会有多个原数据共享同一个 hash code, 也就是哈希冲突


# handle collision
- 哈希碰撞不可避免, 因此需要采取一些办法来处理冲突
- `bucket`, 处理冲突的一种方法就是, 将原来数组中的只能存储一个数据的方格改成桶 `bucket`, 桶的意思就是可能可以存储多个数据
	- `LinkedList bucket`
	- `ArrayList bucket`
	- `ArraySet bucket`
- 这种方法也称为拉链法
![[Pasted image 20240707173732.png]]
![[Pasted image 20240707173949.png]]


# resizing
- 如果桶的数量是固定的, 那么随着数据量的增加, 哈希表还是会退化成链表
- 因此需要设置某种阈值, 在超过阈值时进行桶扩容
- `N`, 元素个数
- `M`, 桶个数
- `N/M`, 理想情况下每个桶上装载的元素个数, 也成为负载因子
- 设定某个阈值负载因子, 超过之后就将桶数量翻倍 (可以选择其他因子, 但是翻倍是最佳实践), 然后再将元素再分散 ![[Pasted image 20240707174742.png]]


# hash function
- 为了让上面的策略尽可能生效, 还需要一个优质的函数来生成 `hash code`, 这个函数称为 `hash function`
- `hash function` 生成的数据应该尽量分散
- java 中, `Object` 类有一个默认的 `hashCode()` 方法, 默认返回对象的内存地址
- java 中的 `%` 取模有点反直觉, 因此需要使用一些其他方法例如 `Math.floorMod(-1, 4)` ![[Pasted image 20240707175753.png]]
- 如何写出好的 `hashCode()`: java 标准库中的一个 `String.hashCode()` 例子 ![[Pasted image 20240707180227.png]]
	- 以字符串为例, 通常选择质数作为基数求幂
	- 错开幂次 ![[Pasted image 20240707180625.png]] ![[Pasted image 20240707180639.png]]
