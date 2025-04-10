- [x] Lecture video
- [x] Lecture note

[[10.1 ADTs]]
[[10.2 Binary Search Trees]]
[[3.2 Binary Search Tree]]

# ADT
- Abstract data type
- 它定义了一种抽象数据类型, 类似于接口, 只定义了行为, 没有定义具体的实现
- 例如 `list` 这个 ADT 就可以实现为 `LinkedList, ArrayList`
- `java.util.collections` 提供了很多这样子的 ADT 接口以及一些具体实现
	- `List`: `LinkedList, ArrayList`
	- `Set`: `HashSet, TreeSet`
	- `Map`: `HashMap, TreeMap`


# Trees
- 由一组节点和一组链接这些节点的边组成, 任意两个节点之间最多只有一条路径, 因此不会有循环 (区别于图)
- 除了根节点之外, 其他节点都会有且只有一个父节点


# Binary search trees
- 二叉搜索树
- 是一种递归的数据结构
- 对一个每一个节点, 值 X
- 左子树中的每一个值都小于 X
- 右子树中的每一个值都大于 X
- 效率
	- 矮胖的 BST 查找将花费 `O(logN)` 的时间
	- 瘦高的 BST (退化成链表) 查找将花费 `O(N)` 的时间
- API
	- `get(key)`
	- `insert(key, val)`
	- `delete(key)`

- `get, insert` 都比较相似比较简单, 这里重点看一些 `delete` 的操作
- 根据删除节点的子节点个数, 可以分成三类
	- 0 个子节点, 直接删除, 返回 null, 让它的父节点不再引用它
	- 1 个子节点, 该节点的子节点也一定满足让该节点的父节点合法的条件, 因此直接让父节点引用子节点, 该节点自己不再被引用
	- 2 个子节点, 找到该节点的 `前任或者继任者`, 由于 `前任, 继任者` 的特点, 他们只会有 0 个或 1 个子节点, 因此可以删除 `前任或继任者`, 然后使用 `前任或继任者` 替代该节点![[Pasted image 20240705234110.png]]![[Pasted image 20240705234150.png]]


# TreeSet & TreeMap
- 这两个数据结构的第一眼

- set ![[Pasted image 20240705234344.png]]
- map ![[Pasted image 20240705234356.png]]
