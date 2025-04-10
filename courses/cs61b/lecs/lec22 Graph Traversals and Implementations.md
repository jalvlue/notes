- [x] Lecture video
- [x] Lecture note
- [ ] Reading ch 18


# Graph traversal overview
- `DFS Preorder`
- `DFS Postorder`
- `BFS`


# Breadth first search
- BFS 跟 DFS 不同, 迭代遍历一般是更好的实现
- 使用队列
- 在无权图中, BFS 将可以找到两个节点之间的最短路径
- 寻找无权图的最短路径是 BFS 的一种重要引用

# Graph API
![[Pasted image 20240710094702.png]]

# Graph implementations
- `adjacency matrix`: 邻接矩阵, 使用节点之间的笛卡尔积布尔列表标识两个节点是否相连![[Pasted image 20240712091529.png]]
- `list of edges`: 边集![[Pasted image 20240712091702.png]]
- `adjacency lists`: 邻接表, 最常见的实现, 看起来有点像一个链式哈希表, 每一个节点都是列表的一个 `entry`, 然后将与他直接相连的节点都放进该节点的 `entry` 中 ![[Pasted image 20240712091719.png]]

- 不同实现的运行效率
![[Pasted image 20240712092134.png]]
