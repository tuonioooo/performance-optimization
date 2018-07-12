# **数据库存储的引擎分析**

## 概述

&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;  插件式存储引擎是 MySQL 数据库最重要的特性之一，用户可以根据应用的需要选择如何存储和索引数据、是否使用事务等。MySQL 默认支持多种存储引擎，以适用于不同领域的数据库应用需要，用户可以通过选择使用不同的存储引擎提高应用的效率，提供灵活的存储，用户甚至可以按照自己的需要定制和使用自己的存储引擎，以实现最大程度的可定制性。MySQL 5.0 支持的存储引擎包括 MyISAM、InnoDB、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、ARCHIVE、CSV、BLACKHOLE、FEDERATED 等，其中 InnoDB 和 BDB 提供事务安全表，其他存储引擎都是非事务安全表。可以先查看一下当前数据库可以使用的引擎。

## 存储引擎概念



