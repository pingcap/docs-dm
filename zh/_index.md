---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据迁移任务管理工具，支持从与 MySQL 协议兼容的数据库（MySQL、MariaDB、Aurora MySQL）到 TiDB 的数据迁移。DM 工具旨在降低数据迁移的运维成本。

DM 最新稳定版本是 2.0，相比 1.0 版本支持了以下特性：

- [数据迁移任务的高可用](dm-arch.md#高可用)，部分 DM-master、DM-worker 节点异常后仍能保证数据迁移任务的正常运行。
- [乐观协调模式下的 sharding DDL](feature-shard-merge-optimistic.md) 可以在部分场景下减少 sharding DDL 同步过程中的延迟、支持上游数据库灰度变更等场景。
- 更好的易用性，包括新的[错误处理机制](handle-failed-ddl-statements.md)及更清晰易读的错误信息与错误处理建议。
- 与上下游数据库及 DM 各组件间连接的 [TLS 支持](enable-tls.md)。
- 实验性地支持从 MySQL 8.0 迁移数据。

目前 DM 工具没有图形化管理界面，对大量数据迁移任务管理的支持有限。**如果您需要操作大量数据迁移任务，请谨慎选择使用 DM。**

<NavColumns>
<NavColumn>
<ColumnTitle>关于 TiDB Data Migration</ColumnTitle>

- [什么是 DM？](overview.md)
- [DM 架构](overview.md)
- [性能数据](benchmark-v2.0-ga.md)

</NavColumn>

<NavColumn>
<ColumnTitle>快速上手</ColumnTitle>

- [快速上手试用](quick-start-with-dm.md)
- [使用 TiUP 部署集群](deploy-a-dm-cluster-using-tiup.md)
- [创建数据源](quick-start-create-source.md)
- [创建数据迁移任务](quick-create-migration-task.md)

</NavColumn>

<NavColumn>
<ColumnTitle>部署使用</ColumnTitle>

- [软硬件要求](hardware-and-software-requirements.md)
- [使用 TiUP 部署集群（推荐）](deploy-a-dm-cluster-using-tiup.md)
- [使用 TiUP 离线镜像部署集群](deploy-a-dm-cluster-using-tiup-offline.md)
- [使用 Binary 部署集群](deploy-a-dm-cluster-using-binary.md)
- [使用 DM 迁移数据](migrate-data-using-dm.md)
- [测试 DM 性能](performance-test.md)

</NavColumn>

<NavColumn>
<ColumnTitle>运维操作</ColumnTitle>

- [使用 TiUP 运维集群（推荐）](maintain-dm-using-tiup.md)
- [使用 dmctl 运维集群](dmctl-introduction.md)
- [升级版本](manually-upgrade-dm-1.0-to-2.0.md)
- [手动处理 Sharding DDL Lock](manually-handling-sharding-ddl-locks.md)
- [处理告警](handle-alerts.md)
- [日常巡检](daily-check.md)

</NavColumn>

<NavColumn>
<ColumnTitle>使用场景</ColumnTitle>

- [从 Aurora 迁移数据到 TiDB](migrate-from-mysql-aurora.md)
- [变更同步的 MySQL 实例](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>参考指南</ColumnTitle>

- [DM 架构](dm-arch.md)
- [DM 命令行参数](command-line-flags.md)
- [配置概述](config-overview.md)
- [监控指标](monitor-a-dm-cluster.md)
- [告警信息](alert-rules.md)
- [错误码](error-handling.md#常见故障处理方法)

</NavColumn>

</NavColumns>
