---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
aliases: ['/docs-cn/tidb-data-migration/dev/']
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据迁移任务管理平台，支持从 MySQL 或 MariaDB 到 TiDB 的全量数据迁移和增量数据复制。使用 DM 工具有利于简化错误处理流程，降低运维成本。

DM 2.0 相比于 1.0，主要有以下改进：

- [数据迁移任务的高可用](overview.md#高可用)，部分 DM-master、DM-worker 节点异常后仍能保证数据迁移任务的正常运行。
- [乐观协调模式下的 sharding DDL](feature-shard-merge-optimistic.md) 可以在部分场景下减少 sharding DDL 同步过程中的延迟、支持上游数据库灰度变更等场景。
- 更好的易用性，包括新的[错误处理机制](handle-failed-sql-statements.md)及更清晰易读的错误信息与错误处理建议。
- 与上下游数据库及 DM 各组件间连接的 [TLS 支持](enable-tls.md)。
- 实验性地支持从 MySQL 8.0 迁移数据。

<NavColumns>
<NavColumn>
<ColumnTitle>关于 TiDB Data Migration</ColumnTitle>

- [性能数据](benchmark-v2.0-ga.md)
- [Table routing](key-features.md#table-routing)
- [Block & Allow Lists](key-features.md#block--allow-table-lists)
- [Binlog Event Filter](key-features.md#binlog-event-filter)
- [Online-ddl-scheme](feature-online-ddl-scheme.md)
- [分库分表合并迁移](feature-shard-merge.md)

</NavColumn>

<NavColumn>
<ColumnTitle>快速上手</ColumnTitle>

- [应用场景](scenarios.md)
- [部署集群](quick-start-with-dm.md)
- [迁移任务](migrate-data-using-dm.md)

</NavColumn>

<NavColumn>
<ColumnTitle>部署使用</ColumnTitle>

- [软硬件要求](hardware-and-software-requirements.md)
- [使用 TiUP 部署集群](deploy-a-dm-cluster-using-tiup.md)
- [使用 TiUP 离线镜像部署集群](deploy-a-dm-cluster-using-tiup-offline.md)
- [使用 Binary 部署集群](deploy-a-dm-cluster-using-binary.md)
- [使用 DM 迁移数据](migrate-data-using-dm.md)
- [监控与告警设置](monitor-a-dm-cluster.md)
- [性能测试](performance-test.md)

</NavColumn>

<NavColumn>
<ColumnTitle>运维操作</ColumnTitle>

- [版本升级](manually-upgrade-dm-1.0-to-2.0.md)
- [集群操作](maintain-dm-using-tiup.md)
- [任务管理](dmctl-introduction.md)
- [手动处理 Sharding DDL Lock](manually-handling-sharding-ddl-locks.md)
- [告警处理](handle-alerts.md)
- [日常巡检](daily-check.md)

</NavColumn>

<NavColumn>
<ColumnTitle>教程</ColumnTitle>

- [Data Migration 简单使用场景](usage-scenario-simple-migration.md)
- [分库分表合并场景](usage-scenario-shard-merge.md)
- [从兼容 MySQL 的数据库迁移到 TiDB](migrate-from-mysql-aurora.md)
- [分表合并数据迁移最佳实践](shard-merge-best-practices.md)
- [切换 DM-worker 与上游 MySQL 实例的连接](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>参考指南</ColumnTitle>

- [DM 架构](overview.md)
- [DM 命令行参数](command-line-flags.md)
- [配置概述](config-overview.md)
- [监控指标](monitor-a-dm-cluster.md)
- [告警信息](alert-rules.md)
- [错误码](error-handling.md#常见故障处理方法)

</NavColumn>

</NavColumns>
