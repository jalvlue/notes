SQLite 是一个轻量级的关系数据库管理系统（RDBMS），它具有以下几个主要特点：

### 1. **内嵌数据库**
- SQLite 是一个自包含、无服务器的数据库引擎，数据库文件以单一文件的形式存在，便于携带和管理。

### 2. **零配置**
- 不需要安装或配置，直接将 SQLite 的库文件集成到应用程序中即可使用。

### 3. **轻量级**
- SQLite 的库文件小，适合嵌入式系统和移动应用，资源占用低。

### 5. **ACID 支持**
- SQLite 支持事务，确保数据的一致性和完整性，支持原子性、一致性、隔离性和持久性（ACID）。

### 6. **SQL 支持**
- 支持标准的 SQL 语法，允许用户通过 SQL 语句进行数据查询和操作。

### 7. **广泛应用**
- 通常用于移动应用（如 Android 和 iOS）、桌面应用、嵌入式系统、浏览器等场景。


### 使用场景
- 适合小型到中型应用、快速原型开发、存储配置和日志数据等。

### 总结
SQLite 是一个灵活、高效且易于使用的数据库解决方案，广泛应用于各种软件开发项目中，特别是在需要简单和轻量级数据库解决方案的场景中。


# 内嵌数据库
SQLite 内嵌数据库具有以下几个显著特点：

### 1. **自包含性**
- SQLite 将整个数据库存储在一个单独的 `.db` 文件中，所有的数据、表结构、索引和元数据都保存在这个文件中。这种设计使得数据库的管理和备份变得简单。

### 2. **无服务器架构**
- SQLite 是一个无服务器的数据库，不需要一个独立的数据库服务器进程。应用程序可以直接访问文件，因此不需要网络通信，降低了复杂性。

### 3. **轻量级**
- SQLite 的目标是提供一个轻量级的解决方案，特别适合嵌入式系统和移动应用。它的内存占用和文件大小都较小，适合资源受限的环境。

### 4. **简化的配置**
- 由于不需要配置数据库服务器，开发者可以迅速开始使用。只需将 SQLite 库集成到应用中，并创建一个数据库文件即可。

### 5. **高效的 I/O 操作**
- SQLite 在文件操作上进行了优化，直接对文件进行读写，减少了数据传输的延迟。

### 6. **跨平台兼容性**
- 由于数据存储在一个文件中，SQLite 更容易在不同平台之间迁移和共享。

### 7. **事务支持**
- SQLite 完全支持 ACID 特性，确保数据的一致性和完整性。事务可以确保操作要么全部成功，要么全部失败。

### 8. **多用户支持**
- 尽管是一个单文件数据库，但 SQLite 支持多线程访问，允许多个进程同时读取数据。

### 总结
SQLite 的设计目标是简化数据库的使用和管理，使其适合于小型和中型应用程序。其独特的 `.db` 文件结构使开发者能快速构建和部署应用，而无需复杂的设置和维护。

# 常见用法
SQLite 的用法主要包括以下几个方面：

### 1. **安装和配置**
- SQLite 通常以库的形式分发，可以直接下载并集成到项目中。对于大多数语言（如 Python、Java 等），可以通过包管理工具安装对应的 SQLite 库。

### 2. **创建数据库**
- 可以通过命令行或应用程序代码创建数据库文件：
  ```bash
  sqlite3 mydatabase.db
  ```

### 3. **连接数据库**
- 在应用程序中，使用相应语言的 SQLite 库连接到数据库。
  ```python
  import sqlite3
  connection = sqlite3.connect('mydatabase.db')
  ```

### 4. **创建表**
- 使用 SQL 语句创建表：
  ```sql
  CREATE TABLE users (
      id INTEGER PRIMARY KEY,
      name TEXT NOT NULL,
      age INTEGER
  );
  ```

### 5. **插入数据**
- 向表中插入数据：
  ```sql
  INSERT INTO users (name, age) VALUES ('Alice', 30);
  ```

### 6. **查询数据**
- 查询表中的数据：
  ```sql
  SELECT * FROM users;
  ```

### 7. **更新数据**
- 更新已有记录：
  ```sql
  UPDATE users SET age = 31 WHERE name = 'Alice';
  ```

### 8. **删除数据**
- 删除记录：
  ```sql
  DELETE FROM users WHERE name = 'Alice';
  ```

### 9. **事务管理**
- 使用事务来确保数据一致性：
  ```sql
  BEGIN TRANSACTION;
  -- 执行多个操作
  COMMIT;
  ```

### 10. **使用索引**
- 创建索引以提高查询性能：
  ```sql
  CREATE INDEX idx_name ON users (name);
  ```

### 11. **备份和恢复**
- 由于数据库是一个文件，可以直接复制数据库文件进行备份。

### 12. **使用命令行工具**
- SQLite 提供了命令行工具，支持直接交互式执行 SQL 语句：
  ```bash
  sqlite3 mydatabase.db
  ```

### 总结
SQLite 提供了简单易用的接口和丰富的功能，适合各种应用场景。无论是通过命令行、脚本语言还是嵌入式应用，都可以轻松实现数据的存储和管理。


# The Database File - .db 文件格式

[sqlite-database-file-format](https://www.sqlite.org/fileformat.html)

- 一个 `sqlite` 数据库只包含有一个 `.db` 文件, 但是在进行事务的时候, 会创建一个临时的冗余文件来做回滚


### Pages
- 数据库文件按页组织, 一个 `.db` 文件由多个页组成, 一个数据库中所有页的大小都是相同的, 页是数据库引擎将数据从磁盘加载到内存 (buffer pool) 中的最小单位, 页大小由 header 中的 `page size` 定义
- `b-tree page`
	- `table b-tree interior page`: 该页存储聚簇索引 `b-tree` 的中间节点, 不存储实际数据
	- `table b-tree leaf page`: 该页存储聚簇索引 `b-tree` 的叶子节点, 所有表的数据都实际存放在这个位置上
	- `index b-tree interior page`: 存储二级索引 `b-tree` 的中间节点
	- `index b-tree leaf page`: 存储二级索引 `b-tree` 的叶子节点, 其中保存的是聚簇索引 id, 如果没有索引覆盖则会拿着该记录的 id 去聚簇索引拿所有需要的数据, 也就是一次回表
- `freelist page`
	- `freelist trunk page`: 用于管理数据库中未使用的页面（空闲页面）的信息。- 记录一系列空闲页面的起始地址，帮助快速分配和回收页面。
	- `freelist leaf page`: 用于管理数据库中未使用的页面（空闲页面）的信息。记录一系列空闲页面的起始地址，帮助快速分配和回收页面。
- `payload overflow page`: 用于存储超过单个 B-tree 页面容量的数据。当某个数据行的大小超过了所在页面的容量时，数据会被分割并存储在溢出页面中，溢出页面通过指针与主页面关联。
- `pointer map page`: 用于跟踪数据库中各个页面的指针和类型。提供页面的类型信息，帮助快速定位和管理不同类型的页面。
- `lock-byte page`: 用于实现数据库的锁机制。存储锁状态信息，确保在多用户访问时的同步和数据一致性。
- 基本上所有的数据库读写操作都是以页为单位进行的


### The Database Header
- `.db` 文件的前 `100` 个字节(整个数据库第一页的前 100 个字节)属于整个数据库的文件头 `database file header`, 存储了该数据库的元数据 ![[Pasted image 20240722155652.png]]
- 数据库 `.db` 文件存储的是数据的二进制格式, 按照大端方式去存 (最高有效位在前) [[little-endian 和 big-endian]]


### The Lock-Byte Page
...

### The Freelist Pages
- 软删除的时候, 可能不会立即将该页删除, 而是标记为未使用, 然后放进 `freelist` 中, 以便后续需要时拿过来用
- `freelist` 的组织格式是 `a linked list of freelist trunk pages`, 每一个 `trunk page` 都包含着多个未使用的 `free page number`

### B-tree Pages
- `sqlite` 使用 `b-tree variant` 来组织数据 (实际上是 `b+tree` 的两个变种)
- 这里的 `b-tree` 实际上只是逻辑上的, 实际上只是利用在 page 中的各种 `offset` 去将一页数据组织成一个逻辑 `b-tree`
- `interior page`: `b+tree` 的中间节点, 不存储实际数据, 只是指向下一级
	- `table interior page`
	- `index interior page`
	- 如果这个中间节点包含 `K` 个 key 的话, 就会有 `K+1` 个指向子页面的指针, 这里的指针实际上只是一个 `32-bit unsigned integer`, 存储该子页面的 `page number`
- `leaf page`
	- `table leaf page`: 聚簇索引叶子节点, 存储记录数据
	- `index leaf page`: 二级索引叶子节点, 存储记录的主键

TODO: 在纸上自己尝试绘制一下四种 B-tree pages 的结构
TODO: 理清楚所有表的源点 `sqlite_schema/sqlite_master` 表的位置
