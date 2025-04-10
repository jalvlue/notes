[[sqlite]]

# Print page size
- Database header 在整个数据库文件的前 100 字节, 也就是第一页的前 100 字节, 大端方式排序
- `page size` 这个字段在 `[16, 17]` 字节, 直接按照大端方式读出这两个字节中存储的值即可


# Print number of tables
- 每一个 `b-tree`, 不论是聚簇还是非聚簇都通过 `root page number` 定位

> The b-tree corresponding to the sqlite_schema table is always a table b-tree and always has a root page of 1. The sqlite_schema table contains the root page number for every other table and index in the database file.

- `sqlite_schema` 表的 `table b-tree` 在 db 文件的第一页中, 也就是说, 第一页除了 database header, 就是这个 `sqlite_schema` 的 `b-tree root page`
- `b-tree page header`, 大端存储
	- 对于中间节点页, 12 B
	- 对于叶子节点页, 8 B
- 


# Print table names
- `record format`
	- `header`
	- `body`


# Count rows in a table
TODO


# Read data from a single column
TODO


# Read data from multiple column
TODO

# Filter data with a WHERE clause
TODO


# Retrieve data using a full-table scan
TODO


# Retirieve data using an index
TODO
