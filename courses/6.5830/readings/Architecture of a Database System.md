# 1. introduction
- DBMS
- RDBMS
- capture the main architectural aspects of modern database management systems
- focus on overall system design and stress issues

### 1.1. relational systems: the life of a query
- RDBMS, one of the most important database system
- mainly focus on RDBMS
- main components of a DBMS
![[Pasted image 20250120203153.png]]
	1. `client communications manager`
	2. `process manager`
	3. `relational query processor`
	4. `transactional storage manager`


# 2. process models
- DBMS is a kind of `mulit-user server` which provide service to multiple users concurrently through network connections, a early decision is to establish the multi-user server model
	- `operating system process`
	- `operating system thread`
	- `lightweight thread package`
	- `DBMSs lightweight thread`
	- `DBMS client`
	- `DBMS worker`

### 2.1. uniprocessors and lightweight threads
- two assumptions that leads to discussion
	- `os thread support`
	- `uniprocessor hardware`
- three DBMS process models under these conditions
	- `process per DBMS worker`
	- `thread per DBMS worker`
	- `process pool`

###### 2.1.1. process per DBMS worker
![[Pasted image 20250120210447.png]]
- fire a os process for a client
- and leave the worker process managed by os

##### 2.1.2. thread per DBMS worker
![[Pasted image 20250120210703.png]]
- multi-threaded DBMS server
- fire a os thread for a client connection

###### 2.1.3. process pool
![[Pasted image 20250120210930.png]]
- fire fixed number of worker os process

###### 2.1.4. shared data and process boundaries
- for all three models above, their overall behavior is read request from client, execute the request, get the desired tuples and send result back to client
- for multiplex purposes, shared data through all workers is more efficient
- for `multi-threaded model`, share data means share a heap data-structure among the same address space between all worker threads, which is easy to achieve
- for `multi-process model`, share data required share memory to share data-structures and state
- major shared data
	- `disk IO buffers`
		1. `database IO requests, the buffer pool`, all persistent database data is staged through the `DBMS buffer pool`, this `buffer pool` is a big shared data-structure to all workers, when a thread needs a page to be read in from the database, it generates an IO request specifying the disk address and ask the `buffer pool` to fetch and place this page
		2. `log IO requests, the log tail`, the database log is some operation records generated for transactions and staged to an in-memory queue that is periodically flushed to the log disk
	- `client communication buffers`, SQL is typically used in a `pull` model, clients consume result tuples from a query cursor by repeatedly issuing the `SQL FETCH` requests, which retrieve one or more tuples per request. `shared client communication socket buffers` as a queue for the tuples it produces.
	- `lock table`, the lock table is shared by all workers and is used by the `Lock Manager`

### 2.2 DBMS threads
