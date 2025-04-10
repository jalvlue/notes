# 主要数据类型

##### 字符型
- VARCHAR(10) 定长的字符型数据，初始化确定大小后未输入的字符为空，最大长度为 10
- CHAR(2) 定长的字符型数据
- VARCHAR 2(20) 变长的字符型数据类型，最大长度为 20，不被初始化的长度限制，输入多少则分配多少

##### 数值型
- INTEGER
- FLOAT
- DOUBLE PRECISION
- NUMERIC(precision, scale) numeric 可用来表示任意精度的整数或小数，由参数控制，precision 表示数据总长度，scale 表示小数的位数

##### 日期型
DATE

##### 布尔型
BOOLEAN


# 简单操作
### `select` 选择语句，返回一个结果表
```SQL TI:"选择列"
select column1, column2, ...
from table_name;
```

```SQL TI:"选择全部"
select *
from tablename;
```

`distinct` 语句去除重复值
```SQL ti:"使用 distinct 只列出不同的值"
select distinct column1, column2, ...
from tablename;
```

`where` 子句提取满足条件的记录
```SQL ti:"按条件查询"
select column1, column2, ...
from tablename
where condition;
```
condition 的构造
1. 值比较
	- =, >, <, >=, <=, !=, <>(表示不等于)
2. 逻辑运算
	- and: 同时满足两个条件( `where col > 10 and col < 20` )
	- or: 满足任意一个条件
	- not: 非( `where not col > 10` )
3. 特殊判断
	- is null: 判断空值( `where col is null` )
	- between ... and ...: 在区间内，包括端点( `where col between 1500 and 2000` )
	- in(列表): 在列表中( `where col in(10, 20, 30)` )
	- like: 模糊判断，需要用到通配符
		- % 替代字符串
		- \_ 替代单个字符
```SQL ti:"where condition"
select *
from tablename
where column1 like 'https%';
--用 % 替代https后的字符串，获取以https开头的记录

select *
from tablename
where column2 like 'G_o_le';
--用 _ 代替单个字符
```

`order by` 关键字对结果进行排序
```SQL ti:"order by关键字"
--默认为升序排序
select *
from tablename
order by column1;

select *
from tablename
order by column1 ASC|DESC, column2 ASC|DESC, ...;
```

`rownum` 关键字对返回结果的个数进行限制
```SQL ti:"rownum"
select *
from tablename
where rownum <= 5;
--用rownum限制返回前5个记录
```


### `insert into` 插入语句，向表中插入记录
```SQL ti:"insert into"
--1.不提供列名，按创表时默认顺序插入
insert into tablename
values(value1, value2, ...);

--2.按对应列名插入
insert into tablename(column1, column2, ...)
values(value1, value2, ...);
```


### `update` 更新语句，更新已经存在于表中的数据
```SQL ti:"update"
update tablename
set column1 = value1, column2 = value2, ...
where condition;
```


### `delete` 删除语句，删除表中的记录
```SQL ti:"delete"
delete from tablename
where condition;
```


### `join` 连接语句，基于表之间的共同列将多个表连接起来，方便同时查询不同表中的记录
```SQL ti:"join"
select column1, column2, ...
from tablename1
join tablename2
on condition;
--默认为 inner join
```
不同的 `join` 方式
![[Pasted image 20230422132938.png]]

##### `inner join`，从多个表中返回满足 `join` 条件的所有行
```SQL ti:"inner join"
select column1, column2, ...
from table1
inner join table2
on table1.columnname = table2.columnname;
--与无修饰的 join 是相同的，是 join 的默认
```

##### `left join`，返回左表中的所有行(即使没有匹配的行也返回，并将对应的右表属性置为 `null` )
```SQL ti:"left join"
select columnname(s)
from table1
left join table2
on table1.columnname = table2.columnname;
```

##### `right join`，返回右表中的所有行，与 `left join` 类似
```SQL ti:"right join"
select columnname(s)
from table1
right join table2
on table1.columnname = table2.columnname;
```

##### `full outer join`，返回左表和右表所有的行，没有匹配的对应属性置为 `null`，是 `right join` 和 `left join` 的结合
```SQL ti:"full outer join"
select columnname(s)
from table1
full outer join table2
on table1.columnname = table2.columnname;
```

总结：
`A inner join B` 取交集。
`A left join B` 取 A 全部，B 没有对应的值为 null。
`A right join B` 取 B 全部 A 没有对应的值为 null。
`A full outer join B` 取并集，彼此没有对应的值为 null。
对应条件在 **on** 后面填写。


### `union` 操作符，合并多个 `select` 语句的结果集
用 `union` 合并的 `select` 语句必须有相同的列、对应的数据类型、相同的列顺序
```SQL ti:"union"
select columnname(s) from table1
union
select columnname(s) from table2;
-- union 默认选取不同的值(与 distinct 相同，排除相同的记录)
--允许重复的值需要使用 union all 语句
select columnname(s) from table1
union all
select columnname(s) from table2;
```

### `select into 和 insert into select` 语句复制表的数据并插入到新表中
`select into` 要求被插入数据的表不存在，并且在插入的时候自动创建新表
```SQL ti:"select into"
select columnname(s)
into newtable
from tablename;
```
`insert into select` 要求被插入数据的表已经存在
```SQL ti:"insert into select"
insert into table2
select *from table1;
```


### `create table` 语句创建新表
```SQL ti:"create table"
create table tablename
(
	column1 datatype(size),
	column2 datatype(size),
	column3 datatype(size),
	...
);
```


### `constraint` 语句，给表中的属性值加上约束
`constraint` 可以在创建表时加上
```SQL ti:"add constraint when creating table"
create table tablename
(
	column1 datatype(size) constraintname1,
	column2 datatype(size) constraintname2,
	column3 datatype(size) constraintname3,
	...
);
```
也可以在创表后使用 `alter table` 语句增加(不同约束使用的语句不相同)

`constraint` 类型
##### `not null` 约束，强制不接受 `null` 值
增加了 `not null` 约束就必须包含值，
不添加值无法对记录进行插入、更新等
```SQL ti:"constraint not null"
--创建表时约束 not null
create table tablename
(
	column1 datatype(size) not null,
	column2 datatype(size) not null,
	...
);

--给已经存在的表增加约束 not null (使用 modify 关键字)
alter table tablename
modify columnname datatype not null;

--删除 not null 约束
alter table tablename
modify columnname datatype null;
```

##### `unique` 约束，在所有记录中，`unique` 属性不存在重复值
`unique` 约束可以唯一标识表中的每条记录，为记录提供唯一性
```SQL ti:"constraint unique"
create table tablename
(
	id int not null unique,
	name varchar2(10) not null,
	...
);

alter table tablename
add unique(columnname);
--或者
alter table tablename
add constraint constraint_name unique(column1,column2);
--给约束命名，并且增加到多个属性上

--删除 unique 约束
alter table table
drop constraint constraint_name;
```

##### `primary key` 约束，将属性标记为主码，唯一标识表中的每条记录
主码有唯一( `unique` )、非空( `not null` )的特性
因此 `primary key` 隐含地包含了 `unique` 和 `not null` 约束
```SQL ti:"constraint primary key"
create table tablename
(
	column1 datatype(size) primary key,
	...
);
--或者同时定义多个属性为 primary key
create table tablename
(
	column1 datatype(size),
	column2 datatype(size),
	column3 datetype(size),
	...
	constraint pk_name primary key(column1, column2);
);
--此时为创建了一个主键 pk_name，pk_name 的值由 column1 和 column2 组成

alter table tablename
add primary key(columnname);
--或者
alter table tablename
add constraint pk_name primary key(column1, column2);

alter table tablename
drop constraint pk_name;
```

##### `foreign key` 约束，将表中的某个属性指向另一个表的主码(属性值参照另外一个表的主码)
```SQL ti:"constraint foreign key"
create table table1
(
	columnname datatype(size),
	...
	columnname datatype foreign key preferences table2(pref_columnname);
);

alter table table1
add constraint fk_name
foreign key(columnname)
preferences table2(pref_columnname);

alter table table1
drop constraint fk_name;
```

##### `check` 约束，限定属性的值
```SQL ti:"constraint check"
create table tablename
(
	column1 datatype(size) check(...),
	column2 datatype(size),
	...
);
--或者
create table tablename
(
	column1, datatype(size),
	...
	constraint check_name check(...);
);

alter table tablename
add constraint check_name check(...);

alter table tablename
drop constraint check_name;
```


# SQL 视图(Views)
在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表
视图包含行和列，像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。
```SQL ti:"create view"
create view viewname as
select columnname(s)
from tablename
where condition
```

```SQL ti:"update view"
create or replace view viewname as
select columnname(s)
from tablename
where condition
```

```SQL ti:"drop view"
drop view viewname
```