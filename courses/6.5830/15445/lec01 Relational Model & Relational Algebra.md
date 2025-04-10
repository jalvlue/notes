# 1. Databases

database(DB):
存储相互关联数据的某个数据集合(对现实时间的某方面建模)

database management system(DBMS):
管理(增删查改...)database 的软件

flat file:
database 通常被存储为 CSV(comma-separated value)的格式

data model:
描述 database 中的具体数据的概念集合
通常是关系模型(relational), 此外还有 NoSQL (key/value, graph), array/matrix/vectors

# 2. Relational Model

relation:
关系模型将属性联系开来, 并将这些联系称为一个关系(relation)(也就是一个表)
拥有 n 个属性的关系称为 n 元关系(n-ary relation)
例如 3 元关系(id, name, year)

tuple:
关系的一个记录或者说实体就是一个元组(tuple)
例如 (011, "jack", 1988) 是上面例子的一个元组

keys:
	primary key: 主码, 唯一标识一个元组 (记录)
	foreign key: 外码, 标识这个属性映射到别的关系的某个元组

# 3. Relational Algebra

select: 选择
在关系(表中)选中满足条件的某些记录
`SELECT * FROM relation WHERE ...`

projection: 投影
只获得某些记录的某些属性
`SELECT attributes FROM relation WHERE ...`

union: 并(∪)
将两个表中的内容拼接成一个表(重复记录被合并成一条(包括原来表中的重复及记录)), (使用 UNION ALL 合并所有记录(包括重复))
NOTE: 两个表属性必须相同
`(SELECT * FROM R_1) UNION ALL (SELECT * FROM R_2)`

intersection: 交(∩)
将两个表相同的记录提出并合成一个新表返回
NOTE: 两个表属性必须相同
`(SELECT * FROM R_1) INTERSECT (SELECT * FROM R_2)`

difference: 差
获得左表中不是右表记录的剩余记录
NOTE: 两个表属性必须相同
`(SELECT * FROM R) EXCEPT (SELECT * FROM S)`

product: 笛卡尔积
获得一个包含两个表笛卡尔积的新表
`(SELECT * FROM R) CROSS JOIN (SELECT * FROM S)`
`SELECT * FROM R, S`

join: 连接
根据两个表中的某些相同的属性, 将别的属性组合, 返回新表
`SELECT * FROM R JOIN S USING (attribute_1, attribute_2, ...)`

