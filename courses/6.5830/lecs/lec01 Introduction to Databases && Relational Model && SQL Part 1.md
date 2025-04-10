- What is a database?
	- Structed data collection
	- Records
	- Relations
- What is a database management system (DBMS)?
	- Software systems for storing and querying databases
- Why database systems?
	- To represent data and reduce repeat
	- User's perspective: 
		- Modeling data
		- Querying data

# Zoo data model example
![[Pasted image 20240606222415.png]]


- Entities:
	- Animal: name, age, species
	- Zookeeper: name
	- Cages: clean_time, buildings
- Relations:
	- Animals are in 1 cage;  cages have multiple animals, 使用这样一张表来表示两个动物和洞穴两个实体之间的关系(relation) ![[Pasted image 20240608212401.png]]
	- Keepers keep multiple cages, cages kept by multiple keepers, 同样的使用另外一张表来表示管理员和洞穴之间的关系![[Pasted image 20240608212452.png]]
		

Other representation:
树形, 图形表示
![[Pasted image 20240608212520.png]]

大而全的行文件表表示(数据冗余)
![[Pasted image 20240606222508.png]]

# SQL: structured query language
```sql
SELECT field1, …, fieldM
FROM table1, …
WHERE condition1, …;

INSERT INTO table VALUES (field1, …);

UPDATE table SET field1 = X, …
WHERE condition1,…;
```

[[命令式编程vs声明式编程]]
sql 是声明式编程方式

![[Pasted image 20240608212901.png]]
复杂查询: 自连接和嵌套查询

![[Pasted image 20240608213328.png]]
多重复杂查询

# relational model
![[Pasted image 20240608213653.png]]
- Schema
- Rows, records, tuples

![[Pasted image 20240608214059.png]]

![[Pasted image 20240608214222.png]]

![[Pasted image 20240608214235.png]]


