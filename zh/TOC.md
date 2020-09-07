# TiDB Data Migration (DM) 文档

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## 文档目录

+ 概述
  - [DM 架构](overview.md)
  - [迁移功能介绍](overview.md#迁移功能介绍)
  - [使用限制](overview.md#使用限制)
  - [DM-worker 简介](dm-worker-intro.md)
  - [DM Relay Log](relay-log.md)
+ 核心特性
  - [Table Routing](feature-overview.md#table-routing)
  - [Block & Allow Lists](feature-overview.md#block--allow-table-lists)
  - [Binlog Event Filter](feature-overview.md#binlog-event-filter)
  - [迁移延迟监控](feature-overview.md#迁移延迟监控)
  - [Online-ddl-scheme](feature-online-ddl-scheme.md)
  + Shard Support
    - [简介](feature-shard-merge.md)
    - [使用限制](feature-shard-merge.md#使用限制)
    - [手动处理 Sharding DDL Lock](feature-manually-handling-sharding-ddl-locks.md)
+ Benchmark
  - [DM 1.0-GA 性能测试](benchmark-v1.0-ga.md)
+ 使用场景
  - [简单的从库迁移场景](usage-scenario-simple-replication.md)
  - [分库分表合并场景](usage-scenario-shard-merge.md)
  - [分表合并数据迁移最佳实践](shard-merge-best-practices.md)
  - [DM-worker 在上游 MySQL 主从间切换](usage-scenario-master-slave-switch.md)
- [快速上手](get-started.md)
+ 部署
  + 部署 DM 集群
    - [使用 DM-Ansible](deploy-a-dm-cluster-using-ansible.md)
    - [使用 Binary](deploy-a-dm-cluster-using-binary.md)
  + [使用 DM 迁移数据](replicate-data-using-dm.md)
+ 配置
  - [概述](config-overview.md)
  - [DM-master 配置](dm-master-configuration-file.md)
  - [DM-worker 配置](dm-worker-configuration-file.md)
  - [完整 DM-worker 配置](dm-worker-configuration-file-full.md)
  - [任务配置](task-configuration-file.md)
  - [完整任务配置](task-configuration-file-full.md)
+ DM 集群管理
  - [集群操作](cluster-operations.md)
  - [集群升级](dm-upgrade.md)
+ DM 迁移任务管理
  - [管理数据迁移任务](manage-replication-tasks.md)
  - [任务前置检查](precheck.md)
  - [任务状态查询](query-status.md)
  - [跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)
- [监控 DM 集群](monitor-a-dm-cluster.md)
+ 从与 MySQL 兼容的数据库迁移数据
  - [从 MySQL/Amazon Aurora MySQL 迁移数据](migrate-from-mysql-aurora.md)
- [DM Portal](dm-portal.md)
+ 告警处理
  - [告警信息](alert-rules.md)
  - [告警处理](handle-alerts.md)
+ 故障处理
  - [故障及处理方法](error-handling.md)
  - [性能问题及处理方法](handle-performance-issues.md)
- [DM FAQ](faq.md)
+ 版本发布历史
  + v1.0
    - [1.0.6](releases/1.0.6.md)
    - [1.0.5](releases/1.0.5.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.2](releases/1.0.2.md)
- [TiDB DM 术语表](glossary.md)
