# Overview
- 嵌入式的 `kv` 存储器, 就类似是 `mamcached, sqlite`, 直接将数据库作为本地文件, 程序如果需要使用就直接打开

# Readme
- 简单的 api, 简单的使用方式
- `db`, 最顶层的概念, 单一个文件, 打开 db 时使用文件锁吗? 如果文件资源被占用了, 可以打开吗?
- 支持单一读写事务, 多个读事务 (读者写者问题), 如何做到?


# Architecture
- `DB` 最核心的数据结构
- `Bucket` 实际存储数据的结构
- `Tx` 操作数据的核心数据结构
- 

# Storage model


# Commands

