---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
aliases: ['/docs-cn/tidb-data-migration/dev/']
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/tiflow/tree/master/dm) (DM) 是一款便捷的数据迁移工具，支持从与 MySQL 协议兼容的数据库（MySQL、MariaDB、Aurora MySQL）到 TiDB 的数据迁移。DM 工具旨在降低数据迁移的运维成本。

> **注意：**
>
> - DM 的 GitHub 代码仓库已于 2021 年 12 月迁移至 [pingcap/tiflow](https://github.com/pingcap/tiflow/tree/master/dm)。如有任何关于 DM 的问题，请在 `pingcap/tiflow` 仓库提交，以获得后续反馈。
> - 在较早版本中（v1.0 和 v2.0），DM 采用独立于 TiDB 的版本号。从 DM v5.3 起，DM 采用与 TiDB 相同的版本号。DM v2.0 的下一个版本为 DM v5.3。DM v2.0 到 v5.3 无兼容性变更，升级过程与正常升级无差异，仅仅是版本号上的增加。

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
