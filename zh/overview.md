---
title: Data Migration 简介
aliases: ['/docs-cn/tidb-data-migration/dev/overview/','/docs-cn/tools/dm/overview/']
---

# Data Migration 简介

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据迁移任务管理平台，支持从 MySQL 或 MariaDB 到 TiDB 的全量数据迁移和增量数据复制。使用 DM 工具有利于简化错误处理流程，降低运维成本。

DM 2.0 相比于 1.0，主要有以下改进：

- [数据迁移任务的高可用](#高可用)，部分 DM-master、DM-worker 节点异常后仍能保证数据迁移任务的正常运行。
- [乐观协调模式下的 sharding DDL](feature-shard-merge-optimistic.md) 可以在部分场景下减少 sharding DDL 同步过程中的延迟、支持上游数据库灰度变更等场景。
- 更好的易用性，包括新的[错误处理机制](handle-failed-sql-statements.md)及更清晰易读的错误信息与错误处理建议。
- 与上下游数据库及 DM 各组件间连接的 [TLS 支持](enable-tls.md)。
- 实验性地支持从 MySQL 8.0 迁移数据。

> **注意：**
>
> DM 以 SQL 语句的形式将数据迁移到 TiDB 中，因此各个版本的 DM 都分别兼容**所有版本**的 TiDB。在生产环境中，推荐使用 DM 的最新已发布版本。已发布版本的下载方式参见 [DM 下载链接](https://pingcap.com/docs-cn/stable/reference/tools/download/#tidb-dm-data-migration)。

## DM 架构

DM 主要包括三个组件：DM-master，DM-worker 和 dmctl。

![Data Migration architecture](/media/dm-architecture-2.0.png)

### DM-master

DM-master 负责管理和调度数据迁移任务的各项操作。

- 保存 DM 集群的拓扑信息
- 监控 DM-worker 进程的运行状态
- 监控数据迁移任务的运行状态
- 提供数据迁移任务管理的统一入口
- 协调分库分表场景下各个实例分表的 DDL 迁移

### DM-worker

DM-worker 负责执行具体的数据迁移任务。

- 将 binlog 数据持久化保存在本地
- 保存数据迁移子任务的配置信息
- 编排数据迁移子任务的运行
- 监控数据迁移子任务的运行状态

有关于 DM-worker 的更多介绍，详见 [DM-worker 简介](dm-worker-intro.md)。

### dmctl

dmctl 是用来控制 DM 集群的命令行工具。

- 创建、更新或删除数据迁移任务
- 查看数据迁移任务状态
- 处理数据迁移任务错误
- 校验数据迁移任务配置的正确性

### 高可用

当部署多个 DM-master 节点时，所有 DM-master 节点将使用内部嵌入的 etcd 组成集群。该 DM-master 集群用于存储集群节点信息、任务配置等元数据，同时通过 etcd 选举出 leader 节点。该 leader 节点用于提供集群管理、数据迁移任务管理相关的各类服务。因此，若可用的 DM-master 节点数超过部署节点的半数，即可正常提供服务。

当部署的 DM-worker 节点数超过上游 MySQL/MariaDB 节点数时，超出上游节点数的相关 DM-worker 节点默认将处于空闲状态。若某个 DM-worker 节点下线或与 DM-master leader 发生网络隔离，DM-master 能自动将与原 DM-worker 节点相关的数据迁移任务调度到其他空闲的 DM-worker 节点上（若原 DM-worker 节点为网络隔离状态，则其会自动停止相关的数据迁移任务）；若无空闲的 DM-worker 节点可供调度，则原 DM-worker 相关的数据迁移任务将无法进行。

## 迁移功能介绍

下面简单介绍 DM 数据迁移功能的核心特性。

### Table routing

[Table Routing](key-features.md#table-routing) 是指将上游 MySQL 或 MariaDB 实例的某些表迁移到下游指定表的路由功能，可以用于分库分表的合并迁移。

### Block & allow table lists

[Block & Allow Lists](key-features.md#block--allow-table-lists) 是指上游数据库实例表的黑白名单过滤规则。其过滤规则类似于 MySQL `replication-rules-db`/`replication-rules-table`，可以用来过滤或只迁移某些数据库或某些表的所有操作。

### Binlog event filter

[Binlog Event Filter](key-features.md#binlog-event-filter) 是比库表迁移黑白名单更加细粒度的过滤规则，可以指定只迁移或者过滤掉某些 `schema`/`table` 的指定类型的 binlog events，比如 `INSERT`，`TRUNCATE TABLE`。

### Shard support

DM 支持对原分库分表进行合库合表操作，但需要满足一些使用限制，详细信息请参考[悲观模式分库分表合并迁移使用限制](feature-shard-merge-pessimistic.md#使用限制)和[乐观模式分库分表合并迁移使用限制](feature-shard-merge-optimistic.md#使用限制)。

## 使用限制

在使用 DM 工具之前，需了解以下限制：

+ 数据库版本

    - 5.5 < MySQL 版本 < 8.0
    - MariaDB 版本 >= 10.1.2

    > **注意：**
    >
    > 如果上游 MySQL/MariaDB server 间构成主从复制结构，则需要 5.7.1 < MySQL 版本 < 8.0 或者 MariaDB 版本大于等于 10.1.3。

    在使用 dmctl 启动任务时，DM 会自动对任务上下游数据库的配置、权限等进行[前置检查](precheck.md)。

+ DDL 语法

    - 目前，TiDB 部分兼容 MySQL 支持的 DDL 语句。因为 DM 使用 TiDB parser 来解析处理 DDL 语句，所以目前仅支持 TiDB parser 支持的 DDL 语法。详见 [TiDB DDL 语法支持](https://pingcap.com/docs-cn/dev/reference/mysql-compatibility/#ddl)。

    - DM 遇到不兼容的 DDL 语句时会报错。要解决此报错，需要使用 dmctl 手动处理，要么跳过该 DDL 语句，要么用指定的 DDL 语句来替换它。详见[如何处理不兼容的 DDL 语句](faq.md#如何处理不兼容的-ddl-语句)。

+ 分库分表

    - 如果业务分库分表之间存在数据冲突，可以参考[自增主键冲突处理](shard-merge-best-practices.md#自增主键冲突处理)来解决；否则不推荐使用 DM 进行迁移，如果进行迁移则有冲突的数据会相互覆盖造成数据丢失。
    - 关于分库分表合并场景的其它限制，参见[悲观模式下分库分表合并迁移使用限制](feature-shard-merge-pessimistic.md#使用限制)以及[乐观模式下分库分表合并迁移使用限制](feature-shard-merge-optimistic.md#使用限制)。

+ 操作限制

    - 在一些情况下，DM-worker 重启后不能自动恢复 DDL lock 同步，需要手动处理。详见[手动处理 Sharding DDL Lock](manually-handling-sharding-ddl-locks.md)。

+ DM-worker 切换 MySQL

    - 当 DM-worker 通过虚拟 IP（VIP）连接到 MySQL 且要切换 VIP 指向的 MySQL 实例时，DM 内部不同的 connection 可能会同时连接到切换前后不同的 MySQL 实例，造成 DM 拉取的 binlog 与从上游获取到的其他状态不一致，从而导致难以预期的异常行为甚至数据损坏。如需切换 VIP 指向的 MySQL 实例，请参考[虚拟 IP 环境下的上游主从切换](usage-scenario-master-slave-switch.md#虚拟-ip-环境下切换-dm-worker-与-mysql-实例的连接)对 DM 手动执行变更。
