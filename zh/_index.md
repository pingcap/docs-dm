---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
aliases: ['/docs-cn/tidb-data-migration/dev/']
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/ticdc/tree/master/dm) (DM) 是一款便捷的数据迁移工具，支持从与 MySQL 协议兼容的数据库（MySQL、MariaDB、Aurora MySQL）到 TiDB 的数据迁移。DM 工具旨在降低数据迁移的运维成本。

DM 最新稳定版本是 5.3，相比 2.0 版本支持了以下特性：

- 合并单行数据的多次变更，将点查更新合并为批量操作，实现以更低的延迟将数据从 MySQL 同步数据到 TiDB。
- 增加 DM OpenAPI 以更方便地管理集群（实验特性）。

**重要说明**

- 由于某些原因，DM 的 github 仓库已被移到 [https://github.com/pingcap/ticdc/tree/master/dm](https://github.com/pingcap/ticdc/tree/master/dm)。请到新的仓库提交问题，以获得后续反馈。
- DM 新版本的发布时间表将与 TiCDC 保持一致，因此版本号也将保持一致。自 v2.1.0 起，DM 版本号将改为 v5.3.0。

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
