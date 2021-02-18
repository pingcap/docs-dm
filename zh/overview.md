---
title: Data Migration 简介
aliases: ['/docs-cn/tidb-data-migration/dev/overview/','/docs-cn/tools/dm/overview/']
---

# Data Migration 简介

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据迁移任务管理工具，支持从 MySQL 或 MySQL 协议兼容的数据库（MariaDB、Aurora MySQL）到 TiDB 的数据迁移。使用 DM 工具有利于降低数据迁移的运维成本。

## 核心功能模块

下面简单介绍 DM 数据迁移功能的核心功能模块。

![DM Core Features](/media/dm-core-features.png)

### Block & allow lists

[Block & Allow Lists](key-features.md#block--allow-table-lists) 其过滤规则类似于 MySQL `replication-rules-db`/`replication-rules-table`，被用来过滤掉或指定只迁移某些数据库或某些表的所有操作。

### Binlog event filter

[Binlog Event Filter](key-features.md#binlog-event-filter) 可以被用来过滤掉源数据库的特定表的特定类型的操作，比如过滤掉表 `test`.`sbtest` 的 `INSERT` 操作和库 `test` 下所有表的 `TRUNCATE TABLE` 操作。

### Table routing

[Table Routing](key-features.md#table-routing) 是可以将源数据库的表迁移到下游指定表的路由功能，比如源数据表 `test`.`sbtest1` 的数据同步到 TiDB 的表 `test`.`sbtest2`。它也是分库分表合并迁移所需的一个核心功能。

## 高级特性

### 分库分表合并迁移

DM 支持对源数据的分库分表进行合并迁移，但需要满足一些使用限制，详细信息请参考[悲观模式分库分表合并迁移使用限制](feature-shard-merge-pessimistic.md#使用限制)和[乐观模式分库分表合并迁移使用限制](feature-shard-merge-optimistic.md#使用限制)。

### 对第三方 Online Schema Change 工具变更过程的同步优化

在 MySQL 生态中，gh-ost 与 pt-osc 等工具较广泛地被使用，DM 对其变更过程进行了特殊的优化，以避免对不必要的中间数据进行迁移。详细信息可参考 [online-ddl](key-features.md#online-ddl-工具支持)。

## 使用限制

在使用 DM 工具之前，需了解以下限制：

+ 数据库版本

    - 5.5 < MySQL 版本 < 8.0
    - MariaDB 版本 >= 10.1.2

    > **注意：**
    >
    > 如果上游 MySQL/MariaDB server 间构成主从复制结构，则需要 5.7.1 < MySQL 版本 < 8.0 或者 MariaDB 版本大于等于 10.1.3。

    在使用 dmctl 启动任务时，DM 会自动对任务上下游数据库的配置、权限等进行[前置检查](precheck.md)。

+ DDL 语法兼容性

    - 目前，TiDB 部分兼容 MySQL 支持的 DDL 语句。因为 DM 使用 TiDB parser 来解析处理 DDL 语句，所以目前仅支持 TiDB parser 支持的 DDL 语法。详见 [TiDB DDL 语法支持](https://pingcap.com/docs-cn/dev/reference/mysql-compatibility/#ddl)。

    - DM 遇到不兼容的 DDL 语句时会报错。要解决此报错，需要使用 dmctl 手动处理，要么跳过该 DDL 语句，要么用指定的 DDL 语句来替换它。详见[如何处理不兼容的 DDL 语句](faq.md#如何处理不兼容的-ddl-语句)。

+ 分库分表

    - 如果业务分库分表之间存在数据冲突，可以参考[自增主键冲突处理](shard-merge-best-practices.md#自增主键冲突处理)来解决；否则不推荐使用 DM 进行迁移，如果进行迁移则有冲突的数据会相互覆盖造成数据丢失。
    - 分库分表 DDL 同步限制，参见[悲观模式下分库分表合并迁移使用限制](feature-shard-merge-pessimistic.md#使用限制)以及[乐观模式下分库分表合并迁移使用限制](feature-shard-merge-optimistic.md#使用限制)。

+ 同步的 MySQL 实例变更

    - 当 DM-worker 通过虚拟 IP（VIP）连接到 MySQL 且要切换 VIP 指向的 MySQL 实例时，DM 内部不同的 connection 可能会同时连接到切换前后不同的 MySQL 实例，造成 DM 拉取的 binlog 与从上游获取到的其他状态不一致，从而导致难以预期的异常行为甚至数据损坏。如需切换 VIP 指向的 MySQL 实例，请参考[虚拟 IP 环境下的上游主从切换](usage-scenario-master-slave-switch.md#虚拟-ip-环境下切换-dm-worker-与-mysql-实例的连接)对 DM 手动执行变更。
