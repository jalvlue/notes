- [x] Lecture video
- [x] Lecture note

[[11.4 Rotating Trees]]
[[11.5 Red-Black Trees]]

[[3.3 Balanced Search Trees#red-black BSTs]]


# tree rotations
- B 树很好, 但是实现起来很复杂
- 因此可以在物理上使用 BST 存储, 但是逻辑上是一个 B 树, 也就是红黑树
- 通过节点旋转来达到这样的效果
	- 左旋 ![[Pasted image 20240706165752.png]]
	- 右旋 ![[Pasted image 20240706165848.png]]
- 总的来说, 对于一个 BST, 进行旋转的目的就是让他更加平衡![[Pasted image 20240706170110.png]]![[Pasted image 20240706170325.png]]

# left leaning red black tree
- LLRB
- 在逻辑上是一个 B 树
- 在物理上是一个 BST
- LLRB 红黑树就是这种结构的一个实现, 使用红边将逻辑上同一个在同一个节点的值链接起来 ![[Pasted image 20240706170747.png]]
- 物理上的 BST 可能不平衡, 但是也非常茂盛, 同时值得一提的是, 从根节点到所有叶子节点, 路径中包括的黑边个数都相等 (来自于 B 树平衡的不变性)
- LLRB 实现, 具体有三个规则, 以 `2-3 tree` 为例
	- 遇到三个连续的左倾红边节点时, 先右旋获得结构良好的节点, 然后颜色翻转 `rotateRight + colorFlip` ![[Pasted image 20240706174454.png]]
	- `rotateLeft`
	- 具体到代码: ![[Pasted image 20240706174727.png]]

# LLRB operations
- `rotateLeft(node)`
- `rotateRight(node)`
- `colorFlip(node)`
- `insert-O(logN)`
- `get-O(logN)`
