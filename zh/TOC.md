# TiDB Data Migration (DM) 文档

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## 文档目录

+ 关于 DM
  + [什么是 DM？](overview.md)
  + 基本功能
    - [Table routing](key-features.md#table-routing)
    - [Block & Allow Lists](key-features.md#block--allow-table-lists)
    - [Binlog Event Filter](key-features.md#binlog-event-filter)
  + 高级功能
    + 分库分表合并迁移
      - [概述](feature-shard-merge.md)
      - [悲观模式](feature-shard-merge-pessimistic.md)
      - [乐观模式](feature-shard-merge-optimistic.md)
    - [迁移使用 GH-ost/PT-osc 的源数据库](feature-online-ddl-scheme.md)
  + [DM 架构](dm-arch.md)
  + [性能数据](benchmark-v2.0-ga.md)
- [快速上手](quick-start-with-dm.md)
+ 部署使用
  - [软硬件要求](hardware-and-software-requirements.md)
  + 部署 DM 集群
    - [使用 TiUP（推荐）](deploy-a-dm-cluster-using-tiup.md)
    - [使用 TiUP 离线镜像](deploy-a-dm-cluster-using-tiup-offline.md)
    - [使用 Binary](deploy-a-dm-cluster-using-binary.md)
  + [使用 DM 迁移数据](migrate-data-using-dm.md)
  + [测试 DM 性能](performance-test.md)
+ 运维操作
  + 集群运维工具
    - [使用 TiUP 运维集群（推荐）](maintain-dm-using-tiup.md)
    - [使用 dmctl 运维集群](dmctl-introduction.md)
  + 升级版本
    - [1.0.x 到 2.0.x 手动升级](manually-upgrade-dm-1.0-to-2.0.md)
    - [1.0.x 版本间升级](upgrade-dm-1.0.md)
  - [管理数据源配置](manage-source.md)
  + 管理迁移任务
    - [任务配置向导](task-configuration-guide.md)
    - [任务前置检查](precheck.md)
    - [创建任务](create-task.md)
    - [查询状态](query-status.md)
    - [暂停任务](pause-task.md)
    - [恢复任务](resume-task.md)
    - [停止任务](stop-task.md)
    - [处理出错的 DDL 语句](handle-failed-ddl-statements.md)
  - [手动处理 Sharding DDL Lock](manually-handling-sharding-ddl-locks.md)
  - [管理迁移表的表结构](manage-schema.md)
  - [处理告警](handle-alerts.md)
  - [日常巡检](daily-check.md)
+ 使用场景
  - [简单数据迁移场景](usage-scenario-simple-migration.md)
  - [分库分表合并场景](usage-scenario-shard-merge.md)
  - [从 Aurora 迁移数据到 TiDB](migrate-from-mysql-aurora.md)
  - [TiDB 表结构存在更多列的迁移场景](usage-scenario-downstream-more-columns.md)
  - [变更同步的 MySQL 实例](usage-scenario-master-slave-switch.md)
+ 故障处理
  - [故障及处理方法](error-handling.md)
  - [性能问题及处理方法](handle-performance-issues.md)
+ 性能调优
  - [配置调优](tune-configuration.md)
+ 参考指南
  + 架构
    - [DM 架构简介](dm-arch.md)
    - [DM-worker 简介](dm-worker-intro.md)
  - [DM 命令行参数](command-line-flags.md)
  + 配置
    - [概述](config-overview.md)
    - [DM-master 配置](dm-master-configuration-file.md)
    - [DM-worker 配置](dm-worker-configuration-file.md)
    - [上游数据库配置](source-configuration-file.md)
    - [数据迁移任务配置向导](task-configuration-guide.md)
  + 安全
    - [为 DM 的连接开启加密传输](enable-tls.md)
    - [生成自签名证书](generate-self-signed-certificates.md)
  - [监控指标](monitor-a-dm-cluster.md)
  - [告警信息](alert-rules.md)
  - [错误码](error-handling.md#常见故障处理方法)
+ [常见问题](faq.md)
+ [术语表](glossary.md)
+ 版本发布历史
  + v2.0
    - [2.0.1](releases/2.0.1.md)
    - [2.0 GA](releases/2.0.0-ga.md)
    - [2.0.0-rc.2](releases/2.0.0-rc.2.md)
    - [2.0.0-rc](releases/2.0.0-rc.md)
  + v1.0
    - [1.0.6](releases/1.0.6.md)
    - [1.0.5](releases/1.0.5.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.2](releases/1.0.2.md)
