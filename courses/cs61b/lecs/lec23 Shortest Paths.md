- [x] Lecture video
- [x] Lecture note
- [ ] Reading ch19


# Shortest paths
- 最短路径, 也就是两个节点之间, 在衡量框架下最短的那一条路径 (如果有)
- 因此在最短路中, 衡量框架的选择非常重要, 也就是图中, 边的权值
- 对于无权图, BFS 可以获得最短路径


# Dijktra's algorithm and single-source shortest paths
- 单源最短路问题: 寻找从一个节点出发, 到其他所有节点的最短路径
- 如果是无权图, 直接使用 BFS 计算边的数量即可
- 如果是有权图, 可以使用迪杰斯特拉算法
- Dijkstra 算法的结果永远是一个树![[Pasted image 20240716090512.png]]
- 因此, 在 V 个节点的图中, Dijkstra 算法得到的单源最短路径永远有 V-1 条边
- `O(E logV)`
- 对于有负权值的图, Dijkstra 算法会失效, 此时可能需要 Bellman-Ford 算法


### 算法实现
- 两个数组
	- `distTo[]`, 存储当前节点到源节点的距离
	- `edgeTo[]`, 存储在当前距离的路径(curr->source)下, 当前节点的上一个节点
- 从源点出发, 将直接相连的节点的 `distTo, edgeTo` 设置好
- 在已设置的 `distTo` 中找到最小的那个节点 (数学证明这个节点的路径已经是最短的了)
- 将与最小节点直接相连的 `distTo, edgeTo` 设置好, 如果有些节点之前已经设置过了, 那么根据 `distTo` 的大小判断取那个路径 (取 `distTo` 小的最短路径)
- 重复以上过程, 直到所有节点的最短路径都确定
- 为了快速访问/删除距离最小的节点 (已经确定了最短路径的节点), 通常使用一个优先队列来维护 `distTo`


# A* single-target shortest paths
[[A_star]]
[[A_star implementation]]

- 启发式求从某一个节点到另一个节点的最短路
- 跟 Dijkstra 相比, 只会考虑一个 `源点-目标` 这一条最短路径, 而不会求其他的最短路径
- 这里使用 `A*` 求解, 只是对 Dijkstra 进行简单的修改 (增加一个启发函数, 跳过一些节点)
- 启发函数的选择很重要, 对最后路径的选择影响非常大


### 算法实现
- 确定一个启发函数 `heuristic(v, goal)`, 将计算出所有路途中所有节点到目标节点的启发函数值
- 然后, 在 Dijkstra 算法的 `distTo[]` 基础上, 再加上这个启发函数值, 选出最短路径中确定的节点
- 重复以上过程, 直到走到 `goal`

