写个 2048

# task 1: public static boolean emptySpaceExists(Board b)
任务一很简单, 就是编写一个 `static` 方法, 检测整个板子中有没有空位, 直接遍历板子, 遇到空的就直接返回 `true`, 没遇到就在最后返回 `false`


# task 2: public static boolean maxTileExists(Board b)
任务二也很简单, 编写一个 `static` 方法, 检测板子中是否存在指定的最大块, 用于判定游戏是否结束。直接遍历整个板子...

# task 3: public static boolean atLeastOneMoveExists(Board b)
任务三是编写一个 `static` 方法，检测是否还能再移动一步，用于判定游戏是否结束。具体想法就是分成三类：
1. 板子里还有空位--一定可以再移动至少一步
2. 板子里没有空位，但是相邻块中，存在值相等的--可以移动合并--一定可以再移动至少一步
3. 板子里没有空位，相邻块也没有值相等的--不能再移动了

具体做法就是：板子有空直接返回，没空则遍历板子，对每一块都获取邻居，邻居中存在值相等的就直接返回

获取邻居这里学到了 java 中动态数组的用法
```java
ArrayList<Tile> res = new ArrayList<>();
...
res.add(neighbor)
...
return res.toArray(new Tile[0]);
```
可以使用 `res.toArray(type)` 返回底层数组

# task 4: public boolean tilt(Side side)
编写每次做出操作后, 板子发生改变的逻辑

这里需要注意几个点：
1. 板子提供了一个特殊的 `view`, 方便
