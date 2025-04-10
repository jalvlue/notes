- [ ] TODO: 本周干完

# overview of gitlet
- 对 `git` 的简单模拟
- 支持的功能:
	- `commit`
	- `checkout`
	- `log`
	- `branch`
	- `merge`
	- `push/pull remote-repo`
- 一个 `commit` 中支持多文件的修改
- 对在某个 `commit` 中被修改的某个文件来说, 一个 `commit` 就是一个版本, 每一个 `commit` 就像是版本链上的一个节点, 因此可以将某个文件的提交记录看作是一个链表, 每一个 `commit` 都是其中一个节点, 并执行它的上一个 `parent-commit` ![[Pasted image 20240713213945.png]]
  - 由于需要提供 `branch` 这个特性, 因此一个提交可能是后面多个提交 (来自不同分治) 的 `parent-commit`, 从而, 此时的提交记录就不再是一个链表, 而更像是一个提交树 ![[Pasted image 20240713214420.png]]
  - `HEAD` 指针将会指向当前激活的分治的最新提交 (如果没有分离 `HEAD` )
  - `commit` 一旦创建, 就不能被修改, 删除, 只能在当前基础上创建新的 `commit`

# internal structures
- `git` 中的所有东西都被称为 `object`, 大概有这几种分类
	- `blobs`: 保存的文件的实际内容, 由于一个文件可能会在多个 `commit` 中被修改, 因此一个文件会对应多个 `blobs`, 每一个都被不同的 `commit` 追踪
	- `trees`: 目录结构, 将目录名映射到 `blob` 或其他子目录树的引用
	- `commits`: 提交的元信息, 一组修改的文件名到对应 `blob` 的映射, 同时还有引用 `tree`, ` parent-commit `
- `SHA-1, cryptographic hash function`, 从任何比特流中 (文件) 生成 `160-bit` 的哈希序列
- 所有的文件 `blobs, commits, trees` 都会保存在项目根目录下的 `.gitlet` 文件夹, 并且, 命名基本上就是该 `object` 的哈希值
- 总的架构图 ![[Pasted image 20240713230921.png]]


# detailed spec of behavior
- `gitlet.Main`
- `gitlet.Commit`
- `gitlet.Repository`
- ... 还有其他 `helper classes`


# The Commands
- 在阅读需要支持的命令时应该想着什么样的数据结构可以支持这样的命令
- 如何复用代码


### init
- `java gitlet.Main init`
- `O(1)`


### add
- `java gitlet.Main add [file name]`
- `O(N), N for the size of file added`

### commit
- `java gitlet.Main commit [message]`
- `O(N), N for number of staging files`

### rm
- `java gitlet.Main rm [file name]`
- `O(1)`


### log
- `java gitlet.Main log`
- `O(N)`
- ![[Pasted image 20240713233943.png]]


### global-log
- `java gitlet.Main global-log`
- `O(N)`


### find
- `java gitlet.Main find [commit message]`
- `O(N)`


### status
- `java gitlet.Main status`
- `O(N)`
```
=== Branches ===
*master
other-branch
  
=== Staged Files ===
wug.txt
wug2.txt
  
=== Removed Files ===
goodbye.txt
  
=== Modifications Not Staged For Commit ===
junk.txt (deleted)
wug3.txt (modified)
  
=== Untracked Files ===
random.stuff
  
```

### checkout
### branch
### rm-branch
### reset
### merge
TODO

# skeleton
- 代码框架基本跟 [[Lab6 Getting Started on Project 2]] 一样


# dealing with files
- `gitlet.Utils`

# serialization details
todo

# coding
