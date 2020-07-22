---
title: DM 配置简介
aliases: ['/docs-cn/dev/reference/tools/data-migration/configure/overview/','/docs-cn/v3.1/reference/tools/data-migration/configure/overview/','/docs-cn/v3.0/reference/tools/data-migration/configure/overview/','/docs-cn/v2.1/reference/tools/data-migration/configure/overview/']
---

# DM 配置简介

本文档简要介绍 DM (Data Migration) 的配置文件和数据同步任务的配置。

## 配置文件

- `inventory.ini`：使用 DM-Ansible 部署 DM 集群的配置文件。需要根据所选用的集群拓扑来进行编辑。详见[编辑 `inventory.ini` 配置文件](deploy-a-dm-cluster-using-ansible.md#第-7-步编辑-inventoryini-配置文件)。
- `dm-master.toml`：DM-master 进程的配置文件，包括 DM 集群的拓扑信息、MySQL 实例与 DM-worker 之间的关系（必须为一对一的关系）。使用 DM-Ansible 部署 DM 集群时，会自动生成 `dm-master.toml` 文件，各项配置说明详见 [DM-master 配置文件介绍](dm-master-configuration-file.md)
- `dm-worker.toml`：DM-worker 进程的配置文件，包括上游 MySQL 实例的配置和 relay log 的配置。使用 DM-Ansible 部署 DM 集群时，会自动生成 `dm-worker.toml` 文件，各项配置说明详见 [DM-worker 配置文件介绍](dm-worker-configuration-file.md)

## 同步任务配置

### 任务配置文件

使用 DM-Ansible 部署 DM 集群时，`<path-to-dm-ansible>/conf` 中提供了任务配置文件模板：`task.yaml.exmaple` 文件。该文件是 DM 同步任务配置的标准文件，每一个具体的任务对应一个 `task.yaml` 文件。关于该配置文件的详细介绍，参见 [任务配置文件](task-configuration-file.md)。

### 创建数据同步任务

用户可以基于 `task.yaml.example` 文件来创建数据同步任务，具体步骤如下：

1. 复制 `task.yaml.example` 为 `your_task.yaml`。
2. 参考[任务配置文件](task-configuration-file.md)来修改 `your_task.yaml` 文件。
3. [使用 dmctl 创建数据同步任务](manage-replication-tasks.md#创建数据同步任务)。

### 关键概念

DM 配置的关键概念如下：

| 概念         | 解释          | 配置文件        |
| :------------ | :------------ | :------------------ |
| source-id  | 唯一确定一个 MySQL 或 MariaDB 实例，或者一个具有主从结构的复制组，字符串长度不大于 32 | `inventory.ini` 的 `source_id`；<br/> `dm-master.toml` 的 `source-id`；<br/> `task.yaml` 的 `source-id` |
| DM-worker ID | 唯一确定一个 DM-worker（取值于 `dm-worker.toml` 的 `worker-addr` 参数） | `dm-worker.toml` 的 `worker-addr`；<br/> dmctl 命令行的 `-worker` / `-w` flag |

### 关闭检查项

DM 默认会对不同任务进行若干检查，用户可以在任务配置文件中使用`ignore-checking-items`配置关闭检查。`ignore-checking-items`是一个列表，其中可能的取值包括：
| 取值   | 含义   |
| :----  | :-----|
| all | 关闭所有检查 |
| dump_privilege | 关闭检查数据库用户是否具有 dump 相关权限 |
| replication_privilege | 关闭检查数据库用户是否具有 replication 相关权限 |
| version | 关闭检查上游数据库版本是否在支持范围内 |
| binlog_enable | 关闭检查上游数据库是否已启用 binlog |
| binlog_format | 关闭检查上游数据库 binlog 格式是否为 row |
| binlog_row_image | 关闭检查上游数据库 binlog_row_image 是否为 FULL|
| table_schema | 关闭检查上游表结构是否支持 |
| schema_of_shard_tables | 关闭检查上游各分表表结构是否一致 |
| auto_increment_ID | 关闭检查上游各分表是否具有自增 ID |
