---
title: Mysql总结
date: 2024-01-08 09:41:54
tags: [Mysql, 八股文]
---

# Mysql 总结

## 基础篇

### Mysql 的一条语句是如何执行的

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240317093516.png)

Server 层是上层，负责建立连接、分析和执行 SQL。底层存储引擎层负责数据的存储和提取，支持 **InnoDB（默认存储引擎）**、MyISAM、Memory 等多个存储引擎。

1. 客户端与 server 层建立 tcp 连接并校验用户名密码
2. 对传过来的 sql 进行解析，词法分析识别关键词，语法分析构建语法树。
   ![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240317094400.png)
3. 预处理 sql，检查表和字段是否存在，将 `select *`中的`*`扩展为全部字段
4. 确定 SQL 语句的执行计划，比如当有多个索引的时候，基于查询成本的考虑来决定选择使用哪个索引，我们可以在查询语句最前面加个 `explain` 命令来输出本条 sql 的执行计划。
   ![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240317095333.png)
5. 执行 sql，下面举几个例子

- 主键等值：把条件交给存储引擎，存储引擎直接返回一条
- 全表扫描：存储引擎返回第一条记录，执行器判断并返回客户端，存储引擎返回下一条记录

### Mysql 一行记录是如何存储的

#### 数据存储在哪个文件

每一个数据库有一个文件夹，当我们给这个数据库创建一个 t_order 表的时候，文件夹中会存在三个文件

- db.opt：用来存储当前数据库的默认字符集和字符校验规则。
- t_order.frm：存放表结构
- t_order.ibd：存放表数据

#### 表空间文件的结构是怎么样的

InnoDB 存储引擎的逻辑存储结构大致如下图：
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240317102558.png)

- 页是 InnoDB 存储引擎磁盘管理的最小单元，页的大小固定 16KB。数据库每次读写都是以 16KB 为单位的，一次最少从磁盘中读取 16K 的内容到内存中，一次最少把内存中的 16K 内容刷新到磁盘中。
- 区可以使得 B+树中每层双向链表相邻的页的物理位置也相邻，这样可以使用顺序 IO 提高速度

#### InnoDB 行格式 COMPACT

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240317103104.png)

- 变长字段长度列表：用来存放当前记录所有变长字段的长度
- NULL 值列表：存放所有可以为 NULL 的字段是否为 NULL，每个字段用 1bit 表示
- row_id：隐式主键，如果建表时存在主键或者不存在主键但是存在非空唯一索引，这个字段就没有用了，否则会设置隐式自增主键
- trx_id：事务 id，表明这条记录是由哪个事务生成的
- roll_pointer：指向 undolog 上一个版本的指针

#### varchar(n) 中 n 最大取值为多少

Mysql 一行记录最大长度为 65535 字节，所有字段的长度 + 变长字段字节数列表所占用的字节数 + NULL 值列表所占用的字节数 <= 65535。

#### 行溢出后，MySQL 是怎么处理的

如果一个数据页存不了一条记录，InnoDB 存储引擎会自动将溢出的数据存放到「溢出页」中。

## 索引篇

索引的定义就是帮助存储引擎快速查找数据的一种数据结构，形象的说就是索引是数据的目录。

所谓的存储引擎，说白了就是如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法。

### 索引的分类
