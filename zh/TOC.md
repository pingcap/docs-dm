# TiDB Data Migration (DM) 文档

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## 文档目录

+ 关于 TiDB Data Migration
  + 基本信息
    - [开源信息说明](licensing.md)
    - [性能数据](performance.md)
  - [主要特性](key-features.md)
  - [应用场景](scenarios.md)
+ 快速上手
  - [部署集群](quick-start-with-dm.md)
  - [同步任务](replicate-data-using-dm.md)
+ 部署文档
  - [软硬件要求](hardware-and-software-requirements.md)
  - [环境与系统配置检查](system-configuration-check.md)
  - [配置拓扑结构](configure-topology.md)
  + 安装与启动
    + 部署社区版本
      + Linux (Redhat/CentOS)
        - [使用 TiUP](deploy-a-dm-cluster-using-tiup.md)
        - [使用 Binary](deploy-a-dm-cluster-using-binary.md)
  + [监控与告警设置](monitor-a-dm-cluster.md)
  + [测试验证](create-task-and-verify.md)
  + [性能测试](benchmark-v1.0-ga.md)
+ 运维操作
  - [版本升级](dm-upgrade.md)
  - [扩缩容](scale-a-dm-cluster.md)
  - [重启集群](cluster-operations.md#重启集群组件)
  + 任务管理 参考 [任务前置检查](precheck.md)，[任务状态查询](query-status.md)，[跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)
    - 创建 [参考](manage-replication-tasks.md##创建数据同步任务)
    - 查询 [参考](manage-replication-tasks.md#查询数据同步任务状态)
    - 暂停 [参考](manage-replication-tasks.md#暂停数据同步任务)
    - 重启
    - 删除
  - [切换 DM-worker 与上游 MySQL 实例的连接](usage-scenario-master-slave-switch.md)
  - [告警处理](handle-alerts.md)
  - [日常巡检](daily-check.md)
+ [DM Portal](dm-portal.md)
+ 故障处理
  - [故障及处理方法](error-handling.md)
  - [性能问题及处理方法](handle-performance-issues.md)
+ 教程
  - [简单的从库同步场景](usage-scenario-simple-replication.md)
  - [分库分表合并场景](usage-scenario-shard-merge.md)
  - [从 Aurora 迁移到 TiDB](migrate-from-mysql-aurora.md)
+ 性能调优指南
  + 软件调优
    - [配置调优](tune-configuration.md) 另可参考 [DM Portal](dm-portal.md)
+ 参考指南
  - [架构](overview.md#dm-架构) 另外可参考 [DM-worker 简介](dm-worker-intro.md)，[DM Relay Log](relay-log.md)
  - [DM 命令行参数](command-line-flags.md)
  - [配置文件](configuration-file.md) 参考 [概述](config-overview.md)，[DM-master 配置](dm-master-configuration-file.md)，[DM-worker 配置](dm-worker-configuration-file.md)，[上游数据库配置](source-configuration-file.md)，[任务配置](task-configuration-file.md)
  - [监控指标](monitor-a-dm-cluster.md)
  - [告警信息](alert-rules.md) [参考](monitor-a-dm-cluster.md)
  - [错误码](error-codes.md)
+ [常见问题](faq.md)
+ 版本发布历史
  + v1.0
    - [1.0.2](releases/1.0.2.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.5](releases/1.0.5.md)
+ [术语表](glossary.md)
