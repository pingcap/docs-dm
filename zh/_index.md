---
title: TiDB Data Migration 用户文档
summary: 了解 TiDB Data Migration 用户文档。
aliases: ['/docs-cn/tidb-data-migration/stable/_index/','/docs-cn/tidb-data-migration/v1.0/_index/']
---

# TiDB Data Migration 用户文档

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) 是一体化的数据同步任务管理平台，支持从 MySQL 或 MariaDB 到 TiDB 的全量数据迁移和增量数据同步。使用 DM 工具有利于简化错误处理流程，降低运维成本。

> **注意：**
>
> DM 以 SQL 语句的形式将数据同步到 TiDB 中，因此各个版本的 DM 都分别兼容**所有版本**的 TiDB。在生产环境中，推荐使用 DM 的最新已发布版本。已发布版本的下载方式参见 [DM 下载链接](https://pingcap.com/docs-cn/stable/reference/tools/download/#tidb-dm-data-migration)。

## DM 架构

DM 主要包括三个组件：DM-master，DM-worker 和 dmctl。

![Data Migration architecture](/media/dm-architecture.png)

### DM-master

DM-master 负责管理和调度数据同步任务的各项操作。

- 保存 DM 集群的拓扑信息
- 监控 DM-worker 进程的运行状态
- 监控数据同步任务的运行状态
- 提供数据同步任务管理的统一入口
- 协调分库分表场景下各个实例分表的 DDL 同步

### DM-worker

DM-worker 负责执行具体的数据同步任务。

- 将 binlog 数据持久化保存在本地
- 保存数据同步子任务的配置信息
- 编排数据同步子任务的运行
- 监控数据同步子任务的运行状态

DM-worker 启动后，会自动同步上游 binlog 至本地配置目录（如果使用 DM-Ansible 部署 DM 集群，默认的同步目录为 `<deploy_dir>/relay_log`）。关于 relay log，详见 [DM Relay Log](relay-log.md)。

### dmctl

dmctl 是用来控制 DM 集群的命令行工具。

- 创建、更新或删除数据同步任务
- 查看数据同步任务状态
- 处理数据同步任务错误
- 校验数据同步任务配置的正确性

## 同步功能介绍

下面简单介绍 DM 数据同步功能的核心特性。

### Table routing

Table routing 是指将上游 MySQL 或 MariaDB 实例的某些表同步到下游指定表的路由功能，可以用于分库分表的合并同步。

### Block & allow table lists

Block & allow table lists 是指上游数据库实例表的黑白名单过滤规则。其过滤规则类似于 MySQL `replication-rules-db`/`replication-rules-table`，可以用来过滤或只同步某些数据库或某些表的所有操作。

### Binlog event filter

Binlog event filter 是比库表同步黑白名单更加细粒度的过滤规则，可以指定只同步或者过滤掉某些 `schema`/`table` 的指定类型的 binlog events，比如 `INSERT`，`TRUNCATE TABLE`。

### Shard support

DM 支持对原分库分表进行合库合表操作，但需要满足一些使用限制。

## 使用限制

在使用 DM 工具之前，需了解以下限制：

+ 数据库版本
+ DDL 语法
+ 分库分表
+ 操作限制
+ DM-worker 切换 MySQL
