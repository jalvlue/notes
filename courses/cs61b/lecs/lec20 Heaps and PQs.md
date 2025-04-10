- [x] Lecture video
- [x] Lecture note

[[13.1 The Priority Queue Interface]]
[[13.2 Heaps]]
[[13.3 Priority Queue implementation]]

[[2.4 Priority Queue]]


# Priority queue
- 以最大优先队列为例, PQ 接口必须至少支持这几个操作
	- `insert`
	- `getMax`
	- `deleteMax` 
![[Pasted image 20240708085908.png]]

# Heaps
- 区别于操作系统进程的堆区, 这里只是一种数据结构
- 分为大根堆和小根堆
- 逻辑上是一个完全二叉树
- 可以实现为树或者数组
- 一般都是使用完全二叉树的数组表示



# Tree representations
- 使用树形表示的堆称为二叉堆
- 以大根堆为例
	- 左右子节点必须小于根节点
	- 所有节点递归满足
	- 是一个完全二叉树, 非常茂盛
	- `insert`: 将新节点放入到完全二叉树的下一个位置, 然后执行上浮操作, 将该节点上浮到满足大根堆性质的位置
	- `delete`: 将完全二叉堆最后一个位置的节点拿上去取代根节点, 此时可能不满足二叉堆的性质, 因此对该节点进行下沉操作, 沉到它该在的位置
- 完全二叉树可以在物理上使用连续数组存储, 按照层序编号
- 此时, 父节点的编号 (在数组中的索引) 刚好是它两个子节点编号除二下取整 `parent(k) = k/2` ![[Pasted image 20240708092953.png]]
- 



# Data structure summary
- 目前为止涉及到的数据结构基本上都围绕着搜索问题, 从数据集中获取特定的数据
- 一些 ADT 和他们可能的实现 ![[Pasted image 20240708093802.png]]
