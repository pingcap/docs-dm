---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据同步任务管理平台，支持从 MySQL 或 MariaDB 到 TiDB 的全量数据迁移和增量数据同步。使用 DM 工具有利于简化错误处理流程，降低运维成本。

<NavColumns>
<NavColumn>
<ColumnTitle>关于 TiDB Data Migration</ColumnTitle>

- [快速上手](get-started.md)
- [DM 架构](overview.md)
- [同步功能介绍](overview.md#同步功能介绍)
- [使用限制](overview.md#使用限制)
- [DM-worker 简介](dm-worker-intro.md)
- [DM Relay Log](relay-log.md)

</NavColumn>

<NavColumn>
<ColumnTitle>使用场景</ColumnTitle>

- [简单的从库同步场景](usage-scenario-simple-replication.md)
- [分库分表合并场景](usage-scenario-shard-merge.md)
- [分表合并数据迁移最佳实践](shard-merge-best-practices.md)
- [DM-worker 在上游 MySQL 主从间切换](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>部署集群</ColumnTitle>

- [使用 DM-Ansible 部署集群](deploy-a-dm-cluster-using-ansible.md)
- [使用 Binary 部署集群](deploy-a-dm-cluster-using-binary.md)
- [使用 DM 同步数据](replicate-data-using-dm.md)

</NavColumn>

<NavColumn>
<ColumnTitle>配置</ColumnTitle>

- [概述](config-overview.md)
- [DM-master 配置](dm-master-configuration-file.md)
- [DM-worker 配置](dm-worker-configuration-file.md)
- [任务配置](task-configuration-file.md)

</NavColumn>

<NavColumn>
<ColumnTitle>管理集群</ColumnTitle>

- [集群操作](cluster-operations.md)
- [集群升级](dm-upgrade.md)
- [集群监控](monitor-a-dm-cluster.md)

</NavColumn>

<NavColumn>
<ColumnTitle>管理同步任务</ColumnTitle>

- [管理数据同步任务](manage-replication-tasks.md)
- [任务前置检查](precheck.md)
- [任务状态查询](query-status.md)
- [跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)

</NavColumn>

</NavColumns>