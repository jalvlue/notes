- [x] Lecture video
- [x] Lecture note

[[9.1 Introduction to Disjoint Sets]]
[[9.2 Quick Find]]
[[9.3 Quick Union]]

# overview
- Disjoint set 也就是不相交集合, 用来解决连通性问题
- 连通性问题
	- `connect(int p, int q)`, 将两个节点联通起来
	- `isConnected(int p, int q)`, 检查两个节点是否联通, 可以是直接相连也可以是间接相连
- 此时, 只考虑两个节点是否相连, 而不考虑两个节点是如何相连的
- 因此可以使用集合, 将相连的所有节点都放进一个集合中, 如果在同一个集合中则相连, 反之则不相连


# Quick union
- 逻辑上使用树的形式组织节点, 物理上使用一个数组, 节点值存储其父节点的值
- `connect(p, q)` 时, 会一直往上找到各自的根节点, 然后将一个根节点的值改为另外一个根节点, 例如将 `q` 的根节点值改为 `q` 的根节点值
- 可能的问题: 树的高度太高, 退化成一个链表
![[Pasted image 20240705091451.png]]


# Weighted quick union
- 相较于上面的直接将一个节点作为另外一个节点的父节点
- 使用 `weighted quick union` 将节点数多的作为父节点, 这样可以尽量控制树的平衡性, 从而达到 `O(logN)` 的时间复杂度

![[Pasted image 20240705092419.png]]
![[Pasted image 20240705092408.png]]



