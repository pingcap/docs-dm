---
title: TiDB Data Migration 版本升级
summary: 了解 TiDB Data Migration 工具的版本变更和升级操作。
aliases: ['/docs-cn/tidb-data-migration/stable/dm-upgrade/','/docs-cn/tidb-data-migration/v1.0/dm-upgrade/','/docs-cn/dev/reference/tools/data-migration/dm-upgrade/','/docs-cn/v3.1/reference/tools/data-migration/dm-upgrade/','/docs-cn/v3.0/reference/tools/data-migration/dm-upgrade/','/docs-cn/v2.1/reference/tools/data-migration/dm-upgrade/']
---

# TiDB Data Migration 版本升级

本文档主要介绍各 TiDB Data Migration (DM) 版本间的升级操作步骤以及各版本的版本信息和主要变更。

> **注意：**
>
> - 若无特殊说明，各版本的升级操作均为从前一个有升级指引的版本向当前版本升级。
> - 若无特殊说明，各升级操作示例均假定已经下载了对应版本的 DM 和 DM-Ansible 且 DM binary 存在于 DM-Ansible 的相应目录中（下载 DM binary 可以参考[更新组件版本](cluster-operations.md#更新组件版本)）。
> - 若无特殊说明，各升级操作示例均假定升级前已停止所有迁移任务，升级完成后手动重新启动所有迁移任务。
> - 以下版本升级指引逆序展示。

## 升级到 v1.0.5

### 版本信息

```bash
Release Version: v1.0.5
Git Commit Hash: a8e9f53f91e29756b09a22cdc37a6a6efcdfe55b
Git Branch: release-1.0
UTC Build Time: 2020-04-27 06:56:31
Go Version: go version go1.13 linux/amd64
```

### 主要变更

- 优化了 `UNIQUE KEY` 对应列含 `NULL` 值时的增量复制速度
- 增加对 TiDB 返回的 `Write conflict`（9007 与 8005）错误的重试
- 修复了全量数据导入过程中有可能触发 `Duplicate entry` 错误的问题
- 修复了全量导入完成后上游无数据写入时可能无法 `stop-task`/`pause-task` 的问题
- 修复 `stop-task` 后监控 metrics 仍有数据显示的问题

### 升级操作示例

1. 下载新版本 DM-Ansible，确认 `inventory.ini` 文件中 `dm_version = v1.0.5`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

## 升级到 v1.0.4

### 版本信息

```bash
Release Version: v1.0.4-1-gd681c67
Git Commit Hash: d681c6731d3432f4d8f38ea651f44d49d6860269
Git Branch: release-1.0
UTC Build Time: 2020-03-16 09:45:29
Go Version: go version go1.13 linux/amd64
```

### 主要变更

- DM Portal 新增英文 UI 的支持
- `query-status` 命令增加 `--more` 参数用于显示完整的迁移状态信息
- 修复到下游 TiDB 连接异常导致迁移暂停后，resume-task 可能无法正常恢复迁移的问题
- 修复 online DDL 执行失败后错误清理了 online DDL meta 信息而导致重启任务后无法继续正确处理 online DDL 迁移的问题
- 修复 `start-task` 异常返回的 `query-error` 可能导致 DM-worker panic 的问题
- 修复 `relay.meta` 写入完成前，DM-worker 进程异常停止，导致重启 DM-worker 时可能无法正确恢复 relay log 文件与 `relay.meta` 的问题

### 升级操作示例

1. 下载新版本 DM-Ansible，确认 `inventory.ini` 文件中 `dm_version = v1.0.4`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

## 升级到 v1.0.3

### 版本信息

```bash
Release Version: v1.0.3
Git Commit Hash: 41426af6cffcff9a325697a3bdebeadc9baa8aa6
Git Branch: release-1.0
UTC Build Time: 2019-12-13 07:04:53
Go Version: go version go1.13 linux/amd64
```

### 主要变更

- dmctl 支持命令式使用
- 支持迁移 `ALTER DATABASE` DDL 语句
- 优化 DM 错误提示信息
- 修复全量导入模块在暂停或退出时 data race 导致 panic 的问题
- 修复对下游进行重试操作时，`stop-task` 和 `pause-task` 可能不生效的问题

### 升级操作示例

1. 下载新版本 DM-Ansible，确认 `inventory.ini` 文件中 `dm_version = v1.0.3`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

> **注意：**
>
> 更新至 DM 1.0.3 版本时，需要确保 DM 所有组件 (dmctl/DM-master/DM-worker) 同时升级。不支持部分组件升级使用。

## 升级到 v1.0.2

### 版本信息

```bash
Release Version: v1.0.2
Git Commit Hash: affc6546c0d9810b0630e85502d60ed5c800bf25
Git Branch: release-1.0
UTC Build Time: 2019-10-30 05:08:50
Go Version: go version go1.12 linux/amd64
```

### 主要变更

- 支持自动为 DM-worker 生成部分配置项，减少人工配置成本
- 支持自动生成 mydumper 库表参数，减少人工配置成本
- 优化 `query-status` 默认输出，突出重点信息
- 直接管理到下游的 DB 连接而不是使用内置连接池，优化 SQL 错误处理与重试
- 修复 DM-worker 进程启动时、执行 DML 失败时可能 panic 的 bug
- 修复执行 sharding DDL（如 `ADD INDEX`）超时后可能造成后续 sharding DDL 无法正确协调的 bug
- 修复了有部分 DM-worker 不可访问时无法 `start-task` 的 bug
- 完善了对 1105 错误的自动重试策略

### 升级操作示例

1. 下载新版本 DM-Ansible, 确认 `inventory.ini` 文件中 `dm_version = v1.0.2`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

> **注意：**
>
> 更新至 DM 1.0.2 版本时，需要确保 DM 所有组件 (dmctl/DM-master/DM-worker) 同时升级。不支持部分组件升级使用。

## 升级到 v1.0.1

### 版本信息

```bash
Release Version: v1.0.1
Git Commit Hash: e63c6cdebea0edcf2ef8c91d84cff4aaa5fc2df7
Git Branch: release-1.0
UTC Build Time: 2019-09-10 06:15:05
Go Version: go version go1.12 linux/amd64
```

### 主要变更

- 修复某些情况下 DM 会频繁重建数据库连接的问题
- 修复使用 `query-status` 时潜在的 panic 问题

### 升级操作示例

1. 下载新版本 DM-Ansible, 确认 `inventory.ini` 文件中 `dm_version = v1.0.1`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

> **注意：**
>
> 更新至 DM 1.0.1 版本时，需要确保 DM 所有组件 (dmctl/DM-master/DM-worker) 同时升级。不支持部分组件升级使用。

## 升级到 v1.0.0-10-geb2889c9 (1.0 GA)

### 版本信息

```bash
Release Version: v1.0.0-10-geb2889c9
Git Commit Hash: eb2889c9dcfbff6653be9c8720a32998b4627db9
Git Branch: release-1.0
UTC Build Time: 2019-09-06 03:18:48
Go Version: go version go1.12 linux/amd64
```

### 主要变更

- 常见的异常场景支持自动尝试恢复迁移任务
- 提升 DDL 语法兼容性
- 修复上游数据库连接异常时可能丢失数据的 bug

### 升级操作示例

1. 下载新版本 DM-Ansible, 确认 `inventory.ini` 文件中 `dm_version = v1.0.0`
2. 执行 `ansible-playbook local_prepare.yml` 下载新的 DM binary 到本地
3. 执行 `ansible-playbook rolling_update.yml` 滚动升级 DM 集群组件
4. 执行 `ansible-playbook rolling_update_monitor.yml` 滚动升级 DM 监控组件

> **注意：**
>
> 更新至 DM 1.0 GA 版本时，需要确保 DM 所有组件 (dmctl/DM-master/DM-worker) 同时升级。不支持部分组件升级使用。

## 升级到 v1.0.0-rc.1-12-gaa39ff9

### 版本信息

```bash
Release Version: v1.0.0-rc.1-12-gaa39ff9
Git Commit Hash: aa39ff981dfb3e8c0fa4180127246b253604cc34
Git Branch: dm-master
UTC Build Time: 2019-07-24 02:26:08
Go Version: go version go1.11.2 linux/amd64
```

### 主要变更

从此版本开始，将对所有的配置进行严格检查，遇到无法识别的配置会报错，以确保用户始终准确地了解自己的配置。

### 升级操作示例

启动 DM-master 或 DM-worker 前，必须确保已经删除废弃的配置信息，且没有多余的配置项，否则会启动失败。可根据失败信息删除多余的配置。
可能遗留的废弃配置：

- `dm-worker.toml` 中的 `meta-file`
- `task.yaml` 中的 `mysql-instances` 中的 `server-id`
