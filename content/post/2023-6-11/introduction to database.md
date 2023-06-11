---
title: "数据库概念总结"
date: 2023-06-11T21:18:13+08:00
draft: false
---

## 概念

**SQL vs NoSQL vs NewSQL**：

SQL数据库：SQL数据库，也称为关系型数据库管理系统（RDBMS），是一种用于存储和操作历史数据的传统数据库类型。在这种系统中，信息按照结构化的方式使用表格或关系进行组织。

NoSQL数据库：NoSQL数据库也称为“非关系型数据库”，它使用键值对、文档、图形数据库或宽列存储等多种数据模型，没有固定的数据模式。NoSQL数据库可以水平扩展，可以在多个服务器上进行扩展，而传统的SQL数据库则通常只能进行纵向扩展，即增加单个服务器的资源。

NewSQL数据库：NewSQL数据库是一种结合了传统SQL数据库的关系数据模型和NoSQL数据库的可扩展性和性能的新型数据库类型。它提供了两种方法的优点，可以实现高性能和水平扩展的关系型数据库。

**BASE vs ACID**：

ACID是原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability）的缩写。这是一组保证数据库事务被可靠地处理的属性。

BASE是基本可用（Basically Available）、软状态（Soft state）和最终一致性（Eventual consistency）的缩写。这是一个数据一致性模型，它优先考虑可用性和分区容错性，而不是严格的一致性。

**OLAP vs OLTP vs HTAP**：

OLAP（Online Analytical Processing）用于支持在线分析处理应用程序，如商业智能、数据挖掘、数据分析等。OLAP通常需要处理大量的数据，而且需要进行复杂的查询和分析操作。OLAP系统通常使用列式存储和向量化查询技术来提高查询性能。

OLTP（Online Transaction Processing）用于支持在线事务处理应用程序，如电子商务、金融交易、库存管理等。OLTP系统需要处理大量的事务，需要保证数据的一致性和可靠性，并且需要支持高并发的访问。

HTAP（Hybrid Transactional and Analytical Processing）是一种将OLTP和OLAP结合起来的数据处理方式。在HTAP系统中，数据可以同时用于事务处理和分析处理。HTAP系统通常使用混合存储引擎，可以在行式存储和列式存储之间灵活切换，以支持不同的应用场景。

**Database vs Data Warehouse vs Data Lake**：

数据库存储应用程序所需的当前数据。

数据仓库存储来自一个或多个系统的当前和历史数据，并采用预定义和固定的模式，使业务分析师和数据科学家能够轻松地分析数据。

数据湖一般以数据的形式存储来自一个或多个系统的当前和历史数据，使业务分析师和数据科学家能够轻松地分析数据。

## 分类

![数据库分类](https://dev-media.amazoncloud.cn/4cbdf0e5acd44b428ecd39e0da9044b1_4.png)

主流的数据库有几十种，这张图描述了当前数据库主流产品定位，通过管理数据量大小和SQL功能强弱把各个产品分为四大类：

- OLTP：在线事务处理平台，一般都是关系型数据库来支撑，常见的数据库有：SQLite、MySQL、PostgreSQL、PolarDB、TiDB、OceanBase
- OLAP：在线分析业务，一般是数据仓库，常见的产品有：Teradata、Clickhouse、Doris、Greenplum、Snowflake、AWS Redshift
- NoSQL：新数据模型，互联网行业用得非常多，代表产品有：Redis、Neo4j、InfluxDB、MongoDB、Cassandra、AWS DynamoDB
- BigData：大数据业务，和数据仓库比较类似，但是更擅长处理大规模数据分析业务，主流产品有：HBase、Hadoop、ElasticSearch、Spark

## 架构

### 外部

我们可以根据数据库节点部署整体架构分为几大种类：

### 单机

计算节点和存储节点一般在同一台机器上，通常存储节点是本地硬盘，如单机版的MySQL、Oracle。单机模式没有高可用保障，常用于临时开发测试或者个人学习场景，生产环境不建议使用。

### 分组

**主从**：

一个主数据库负责所有写操作和一部分读操作，而其他一个或多个从数据库则通过复制主数据库的数据来提供读操作。

**主备**：

一个主数据库负责处理所有写操作和一部分读操作，而集群中一个备用数据库则通过复制数据库（不提供读操作）来保证主数据库挂之后的可用性。

**主主**：

多个数据库都可以处理写操作，而其他数据库则通过复制彼此的数据来提供读操作。

### 分片

分片是将数据库中的数据分为多个片段或分片，每个分片可以单独存储在不同的服务器上。这种技术被广泛应用于分布式数据库中，以提高系统的可扩展性和性能。

### 内部

#### 存储引擎

**HEAP**:

HEAP存储引擎是一种基于内存的存储引擎，它将数据存储在内存中而不是磁盘上，因此具有快速的读写速度。使用HEAP存储引擎的表不具有任何索引或约束，因此适用于临时存储数据或缓存数据。

**B+ TREE**:

B+ TREE存储引擎是一种基于磁盘的存储引擎，它将数据存储在磁盘上，支持高效的查找、插入和删除操作。B+ TREE存储引擎通常用于管理关系型数据库中的索引数据。

**COLUMN STORE**:

COLUMN STORE存储引擎是一种基于列的存储引擎，它将表中的每个列存储在独立的列簇中，以提高查询性能和压缩比率。COLUMN STORE存储引擎通常用于处理大型数据集和分析查询。

**LSM-TREE**:

LSM-TREE存储引擎是一种基于磁盘的存储引擎，它将数据存储在内存中的多个层次结构和磁盘上，以提高写入性能和查询性能。LSM-TREE存储引擎通常用于处理大量写入操作和高吞吐量的查询操作，例如日志记录和分析系统。

![存储引擎比较](https://dev-media.amazoncloud.cn/bc95ea03e3834c8d97a5a02e1bf85434_22.png)

#### 查询引擎

![查询引擎比较](https://dev-media.amazoncloud.cn/160bb688f59a44dcbd07a90ff2285876_55.png)

## 设计

### 索引

### 范式

## 展望

**流式数据库**：

**云原生数据库**：

## 参考

- [程序员必须掌握的数据库原理](https://dev.amazoncloud.cn/column/article/63ef527a65a6d47c5e10b97c)
- [SQL vs NoSQL vs NewSQL: An In-depth Literature Review](https://blog.reachsumit.com/posts/2022/06/sql-nosql-newsql/)
- [Databases vs. Data Warehouses vs. Data Lakes](https://www.mongodb.com/databases/data-lake-vs-data-warehouse-vs-database)
