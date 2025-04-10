- [x] lecture video

[[8.1 Efficient Programming]]
[[8.2 An Introduction to Asymptotic Analysis]]


渐进, 也就开始涉及到了程序时间复杂度衡量

# runtime measurement
运行时间衡量, 
- 物理衡量, 直接计时, 会受到数据量和计算机硬件的影响
- 数学方法, 判断程序运行的渐进时间复杂度

# algorithm scaling
- Scaling 就是数据量大小的拓展
- 一个程序的时间复杂度越优秀, 那么数据量拓展之后, 相较于时间复杂度差的程序, 表现会更好

# simplifying algebraic runtime
- 获得简洁的渐进复杂度 `order of growth`
- 去掉常数, 系数, 只取幂最高的那一项


# big theta - Θ
- O 表示法是渐进上界 `R(N) <= O(f(N))`
- Θ 表示法是渐进下界 `R(N) = Θ(f(N))`

![[Pasted image 20240704201335.png]]

