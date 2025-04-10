- [x] Lecture video

# command line compilation
- Proj 2 GitLet 将会编写一个命令行工具, 实现部分 git 的功能
- java 中, 命令行参数会作为 `String` 被传递到 ` public static void main(String[] args) ` 的 ` args ` 中, 索引从 0 开始

# Git: a command line program
- 版本控制, 直接复制多个备份会造成很大的空间冗余
- 使用 git 进行版本控制, 则会利用到 git 的很多优势
- git 的所有文件都使用 `SHA-1` 进行标识并压缩
- git 对文件进行 `SHA-1` 快照, 并对没有修改的文件保存指向之前快照的链接
- 

# storing commits using serialization
- Java 对象进行序列化处理, 然后再作为文件保存到本地中

