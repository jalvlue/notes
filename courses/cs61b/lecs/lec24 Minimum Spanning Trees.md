- [ ] Lecture video
- [x] Lecture note
- [ ] Reading ch20

# overview
- `Minimum Spanning Trees-MSTs`, 最小生成树, 由一个无向图生成的树
	- 是图的子图
	- 拥有全部节点
	- 拥有部分边
	- 是一棵树
	- 边的权值相加是所有的生成树中最小的
- 应用: 由于这是一颗树, 包含了所有节点, 而且是连通的, 还是权值和最小的
	- 多个城市之间的道路铺设
	- 电路电线设计
	- ...


# cut property
- 边际属性
- 如果把整个图中的节点分成两个集合 `S + T`
	- `S` 是已经确认被收录进 MST 的节点
	- `T` 是其他剩下的待收录的节点
	- 这两个集合节点之间的边称为 `cut edge` 边际边
	- 边际属性: 从所有的 `cut edge` 中取出最小的那个, 这条边所连接的 `T` 中的节点就是 MST 中下一个需要收录的节点


# Prim's Algorithm
- 利用了上述 `cut property`, 将节点分成两个集合, 并根据最短的边不断生长 `S` 集合![[Pasted image 20240717125734.png]]
- 也可以看作是一种 Dijkstra 的变种, `distTo[]` 不再是到源点的距离, 而是到整个 `S` 集合树的距离
- `O(E logV)`


# Kruskal's Algorithm
```sql
Initialize the MST to be empty
Consider each edge e in INCREASING order of weight:
    If adding e to the MST does not result in a cycle, add it to e
```
- 基本上也是类似, 只是采取了另一种视角来生成
- 将所有边按照权值从小到大排序
- 每次取出最小的边, 如果跟已取出的边组合不构成循环, 则可以拿下
- 如果构成循环则放弃该边
- 重复直到获取了 V-1 条边
- 这里循环检测就可以使用最开始学到的 [[lec14 Disjoint Sets]]
- `O(E logE)`

