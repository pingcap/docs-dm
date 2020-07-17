---
title: DM 配置简介
---

# DM 配置简介

本文档简要介绍 DM (Data Migration) 的配置文件和数据同步任务的配置。

## 配置文件

- `dm-master.toml`：DM-master 进程的配置文件，包括 DM-master 的拓扑信息、日志等各项配置。配置说明详见 [DM-master 配置文件介绍](dm-master-configuration-file.md)。
- `dm-worker.toml`：DM-worker 进程的配置文件，包括 DM-worker 的拓扑信息、日志等各项配置。配置说明详见 [DM-worker 配置文件介绍](dm-worker-configuration-file.md)。
- `source.toml`：上游数据库 MySQL/MariaDB 相关配置。配置说明详见[上游数据库配置文件介绍](source-configuration-file.md)。

## 同步任务配置

### 任务配置文件

每一个数据同步任务对应一个 `task.yaml` 文件。关于该配置文件的详细介绍，参见[任务配置文件](task-configuration-file.md)。

### 创建数据同步任务

具体步骤如下：

1. [使用 dmctl 将数据源配置加载到 DM 集群](manage-source.md#加载数据源配置)
2. 参考[任务配置文件](task-configuration-file.md)来创建 `your_task.yaml` 文件。
3. [使用 dmctl 创建数据同步任务](create-task.md)。

### 关键概念

DM 配置的关键概念如下：

| 概念         | 解释          | 配置文件        |
| :------------ | :------------ | :------------------ |
| source-id  | 唯一确定一个 MySQL 或 MariaDB 实例，或者一个具有主从结构的复制组，字符串长度不大于 32 | `source.toml` 的 `source-id`；<br/> `task.yaml` 的 `source-id` |
| DM-master ID | 唯一确定一个 DM-master（取值于 `dm-master.toml` 的 `master-addr` 参数） | `dm-master.toml` 的 `master-addr` |
| DM-worker ID | 唯一确定一个 DM-worker（取值于 `dm-worker.toml` 的 `worker-addr` 参数） | `dm-worker.toml` 的 `worker-addr` |
