
# Architecture
- `class GameState`, 代表游戏状态, 棋盘状态, 总的游戏运行时
- `class Place`, 代表棋盘上的一个格子, 是一个链表结构, 可以放置蚂蚁在上面, 也可以被蜜蜂通行
- `class Hive`, 蜂巢, 蜜蜂出生地, 连接到第一个 `place`
- `class AntHomeBase`, 蚂蚁大本营, 保卫的目的地, 连接到最后一个 `place`
- `class Insect`, 昆虫基类
	- `health`
	- `place`
	- `action`
- `class Ant`, 蚂蚁基类
	- `food_cost`
- `class Bee`, 蜜蜂基类

![[Pasted image 20241217200626.png]]

![[Pasted image 20241217200747.png]]


# Phase I: basic gameplay


# Phase 2: More Ants

