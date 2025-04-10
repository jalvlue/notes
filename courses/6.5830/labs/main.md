# GoDB
- a basic database system implemented in Go
- a simple storage layer, based on `Heap file`
- a `buffer pool` for caching pages and implementation page-level locking for transactions
- a set of operators: `Scan, Filter, Join, Order by, Project`
- a SQL parser
- simple transaction support

# missing parts in GoDB
- recovery
- B+ tree index, only supports sequential scan access methods
- query optimization
- ...

# storage layout
- each table is stored in one file on disk, called a `Heap file`
	- contains unordered collections of records
	- without B+tree index, the only way to access records from a heap file(a table) is to scan from beginning to end, namely `sequential scan` via an iterator
	- each file consists of a number of fixed size `heap pages`
	- each page contains a number of fixed size tuples
	- 