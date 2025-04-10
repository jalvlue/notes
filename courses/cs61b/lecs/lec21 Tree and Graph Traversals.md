- [x] Lecture video
- [x] Lecture note
- [x] Reading ch 17

[[17.1 Tree recap]]
[[17.2 Tree Traversal]]
[[17.3 Graphs]]
[[17.4 Graph Problems]]

# Trees
- 节点集+边集
- 任意两个节点之间只有一条路径
- 没有循环路径而且所有节点都连通的图

# Traversals
- 先序, 在文件系统树形表示上很有用![[Pasted image 20240709091544.png]]
- 中序
- 后序

# Level order traversal
- 层序遍历, 使用队列 FIFO 的性质
- 一次将一层的节点放入队列中

# Depth first traversals-trees
- 深度优先遍历, 也就是先序中序后序的统称

# Graphs
- 跟树相区别, 没有层级结构的概念
- 一组节点集 + 一组边集构成
- 节点之间可能有 0~n 条路径
- 简单图: 没有重复边和自循环的图![[Pasted image 20240709092109.png]]
- 有向图: 边是有方向的
- 无向图: 边没有方向![[Pasted image 20240709092210.png]]


有关图的问题
- `s-t path` 节点 s-t 之间是否存在路径 (是否连通)
- `connectivity` 整个图是否连通
- `biconnectivity` 整个图是否双向连通
- `shortest s-t path` s-t 之间的最短路径
- `cycle detection` 图中是否存在循环
- `Euler tour`, 这个图是不是欧拉图, 也就是在遍历整个图的过程中, 每条边只使用一次
- `Hamilton tour`, 这个图是不是哈密尔顿图, 也就是在遍历整个图的过程中, 每个节点只遍历一次
- `planarity`, 平面性, 能不能把图画到二维纸上但是没有相交的边
- `isomorphism`, 同构性, 两个图是否同构, 也就是将边或节点拉动之后是否能得到一摸一样的图


# Depth first traversal-graphs
- 跟树的遍历非常类似, 只是树是有固定结构的, 左子树完了就是右子树, 不会有重复嵌套无限遍历
- 图是没有固定结构的, 因此需要一个 `set` 来将遍历过的节点记录下来, 以避免重复嵌套循环
- 深搜回溯