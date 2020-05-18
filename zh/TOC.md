# TiDB Data Migration (DM) 文档

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## 文档目录 @张学程 @王相

+ TiDB Data Migration 简介
  + 基本信息
    - 开源
    - 性能数据
  - [主要特性](feature-overview.md) 参考 [Online-ddl-scheme](feature-online-ddl-scheme.md)，和 Shard Support 相关文档（[简介](feature-shard-merge.md)，[使用限制](feature-shard-merge.md#使用限制)，[手动处理 Sharding DDL Lock](feature-manually-handling-sharding-ddl-locks.md)）
  - What's New
  - 应用场景 参考 [分表合并数据迁移最佳实践](shard-merge-best-practices.md)，[DM-worker 在上游 MySQL 主从间切换](usage-scenario-master-slave-switch.md)
  - 荣誉列表
  - 售后支持
+ 快速上手
  - [部署集群](get-started.md)
  - [同步任务](replicate-data-using-dm.md)
+ 部署文档
  - 软硬件要求
  - 环境与系统配置检查
  - 配置拓扑结构
+ 安装与启动
  + 部署社区版本
    + Linux (Redhat/CentOS)
      - 使用 TiUP
      - [使用 Binary](deploy-a-dm-cluster-using-binary.md)
    - 监控与告警设置 [参考](monitor-a-dm-cluster.md)
    - 测试验证
    - [性能测试](benchmark-v1.0-ga.md)
+ 运维操作
  - [版本升级](dm-upgrade.md)
  - 扩缩容
  - [重启集群](cluster-operations.md#重启集群组件)
  + 任务管理 参考 [任务前置检查](precheck.md)，[任务状态查询](query-status.md)，[跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)
    - 创建 [参考](manage-replication-tasks.md##创建数据同步任务)
    - 查询 [参考](manage-replication-tasks.md#查询数据同步任务状态)
    - 暂停 [参考](manage-replication-tasks.md#暂停数据同步任务)
    - 重启
    - 删除
  - 告警处理
  - 日常巡检
+ 故障处理 参考 [故障诊断](troubleshoot-dm.md)，[错误含义](error-system.md)
  - [故障及处理方法](error-handling.md)
  - 性能问题及处理方法
+ 开发者指南
+ 教程
  - [简单的从库同步场景](usage-scenario-simple-replication.md)
  - [分库分表合并场景](usage-scenario-shard-merge.md)
  - [从 Aurora 迁移到 TiDB](migrate-from-mysql-aurora.md)
+ 性能调优指南
  + 软件调优
    - 配置调优 另可参考 [DM Portal](dm-portal.md)
+ 参考指南
  - [架构](overview.md#dm-架构) 另外可参考 [DM-worker 简介](dm-worker-intro.md)，[DM Relay Log](relay-log.md)
  - 命令行参数描述
  - 配置文件描述 参考 [概述](config-overview.md)，[DM-master 配置](dm-master-configuration-file.md)，[DM-worker 配置](dm-worker-configuration-file.md)，[上游数据库配置](source-configuration-file.md)，[任务配置](task-configuration-file.md)
  - [监控指标](monitor-a-dm-cluster.md)
  - 告警信息 [参考](monitor-a-dm-cluster.md)
  - 错误码
+ [常见问题](faq.md)
+ 版本发布历史
  + v1.0
    - [1.0.2](releases/1.0.2.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.5](releases/1.0.5.md)
+ [术语表](glossary.md)
+ 文档指南
