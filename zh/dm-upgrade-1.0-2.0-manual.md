---
title: TiDB Data Migration 1.0.x 到 2.0.x 手动升级
summary: 了解 TiDB Data Migration 工具从 1.0.x 到 2.0.2 的手动升级操作。
---

# TiDB Data Migration 1.0.x 到 2.0.x 手动升级

本文档主要介绍如何手动从 DM v1.0.x 升级到 v2.0.x，主要思路为利用 v1.0.x 时的全局 checkpoint 信息在 v2.0.x 集群中启动一个新的增量数据迁移任务。

> **注意：**
>
> - DM 当前不支持在数据迁移任务处于全量导出或全量导入过程中从 v1.0.x 升级到 v2.0.x。
> - 由于 DM 各组件间用于交互的 gRPC 协议进行了较大变更，因此需确保升级前后 DM 集群各组件（包括 dmctl）使用相同的版本。
> - 由于 DM 集群的元数据存储（如 checkpoint、shard DDL lock 状态及 online DDL 元信息等）发生了较大变更，升级到 v2.0.x 后无法自动复用 v1.0.x 的元数据，因此在执行升级操作前需要确保：
>     - 所有数据迁移任务不处于 shard DDL 协调过程中。
>     - 所有数据迁移任务不处于 online DDL 协调过程中。

## 第 1 步：准备 v2.0.x 的配置文件

### 上游数据库配置文件

在 v2.0.x 中将[上游数据库 source 相关的配置](source-configuration-file.md)从 DM-worker 的进程配置中独立了出来，因此需要根据 [v1.0.x 的 DM-worker 配置](https://docs.pingcap.com/zh/tidb-data-migration/stable/dm-worker-configuration-file) 拆分得到 source 配置。

> **注意：**
>
> 当前从 v1.0.x 升级到 v2.0.x 时，如在 source 配置中启用了 `enable-gtid`，则后续需要通过解析 binlog 或 relay log 文件获取 binlog position 对应的 GTID sets。

#### 从 DM-Ansible 部署的 v1.0.x 升级

如果 v1.0.x 是使用 DM-Ansible 部署的，且假设在 `inventory.ini` 中有如下 `dm_worker_servers` 配置：

```ini
[dm_master_servers]
dm_worker1 ansible_host=172.16.10.72 server_id=101 source_id="mysql-replica-01" mysql_host=172.16.10.81 mysql_user=root mysql_password='VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=' mysql_port=3306
dm_worker2 ansible_host=172.16.10.73 server_id=102 source_id="mysql-replica-02" mysql_host=172.16.10.82 mysql_user=root mysql_password='VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=' mysql_port=3306
```

则可以转换得到如下两个 source 配置文件：

```yaml
# 原 dm_worker1 对应的 source 配置，如命名为 source1.yaml
server-id: 101                                   # 对应原 `server_id`
source-id: "mysql-replica-01"                    # 对应原 `source_id`
from:
  host: "172.16.10.81"                           # 对应原 `mysql_host`
  port: 3306                                     # 对应原 `mysql_port`
  user: "root"                                   # 对应原 `mysql_user`
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 对应原 `mysql_password`
```

```yaml
# 原 dm_worker2 对应的 source 配置，如命名为 source2.yaml
server-id: 102                                   # 对应原 `server_id`
source-id: "mysql-replica-02"                    # 对应原 `source_id`
from:
  host: "172.16.10.82"                           # 对应原 `mysql_host`
  port: 3306                                     # 对应原 `mysql_port`
  user: "root"                                   # 对应原 `mysql_user`
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 对应原 `mysql_password`
```

#### 从 Binary 部署的 v1.0.x 升级

如果 v1.0.x 是使用 Binary 部署的，且对应的 DM-worker 配置如下：

```toml
log-level = "info"
log-file = "dm-worker.log"
worker-addr = ":8262"

server-id = 101
source-id = "mysql-replica-01"
flavor = "mysql"

[from]
host = "172.16.10.81"
user = "root"
password = "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="
port = 3306
```

则可转换得到如下的一个 source 配置文件：

```yaml
server-id: 101                                   # 对应原 `server-id`
source-id: "mysql-replica-01"                    # 对应原 `source-id`
flavor: "mysql"                                  # 对应原 `flavor`
from:
  host: "172.16.10.81"                           # 对应原 `from.host`
  port: 3306                                     # 对应原 `from.port`
  user: "root"                                   # 对应原 `from.user`
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 对应原 `from.password`
```

### 数据迁移任务配置文件

对于[数据迁移任务配置文件](task-configuration-file.md)，v2.0.x 基本与 v1.0.x 保持兼容，后续可简单修改后进行使用。

## 第 2 步：部署 v2.0.x 集群

使用 TiUP 按所需要节点数部署新的 v2.0.x 集群。

## 第 3 步：下线 v1.0.x 集群

如果原 v1.0.x 集群是使用 DM-Ansible 部署的，则[使用 DM-Ansible 下线 v1.0.x 集群](https://docs.pingcap.com/zh/tidb-data-migration/stable/cluster-operations#%E4%B8%8B%E7%BA%BF%E9%9B%86%E7%BE%A4) 。

## 第 4 步：升级数据迁移任务

1. 使用 [operate-source](manage-source.md#加载数据源配置) 命令将 [准备 v2.0.x 的配置文件](#第-1步准备-v2.0.x-的配置文件) 中得到的上游数据库 source 配置加载到 v2.0.x 集群中。

2. 在下游 TiDB 中，从 v1.0.x 的数据迁移任务对应的增量 checkpoint 表中获取到对应的全局 checkpoint 信息。

    - 假设 v1.0.x 的数据迁移配置中未额外指定 `meta-schema`（或指定其值为默认的`dm_meta`），且对应的任务名为 `task_v1`，则对应的 checkpoint 信息在下游 TiDB 的 ``` `dm_meta`.`task_v1_syncer_checkpoint` ``` 表中。
    - 使用以下 SQL 语句分别获取该数据迁移任务对应的所有上游数据库 source 的全局 checkpoint 信息。

        ```sql
        > SELECT `id`, `binlog_name`, `binlog_pos` FROM `dm_meta`.`task_v1_syncer_checkpoint` WHERE `is_global`=1;
        +------------------+-------------------------+------------+
        | id               | binlog_name             | binlog_pos |
        +------------------+-------------------------+------------+
        | mysql-replica-01 | mysql-bin|000001.000123 | 15847      |
        | mysql-replica-02 | mysql-bin|000001.000456 | 10485      |
        +------------------+-------------------------+------------+
        ```

3. 更新 v1.0.x 的数据迁移任务以用于启动新的 v2.0.x 的数据迁移任务。

    - 如 v1.0.x 的数据迁移任务配置文件为 `task_v1.yaml`，则将其复制一份为 `task_v2.yaml`。
    - 对 `task_v2.yaml` 进行以下修改：
        - 将 `name` 修改为一个新的、不存在的名称，如 `task_v2`
        - 将 `task-mode` 修改为 `incremental`
        - 为各 source 设置在 step.2 中获取的全局 checkpoint 作为增量迁移的起始点，如：

            ```yaml
            mysql-instances:
              - source-id: "mysql-replica-01"
                meta:
                  binlog-name: "mysql-bin.000123"    # 对应 checkpoint 信息中的 `binlog_name`，但不包含 `|000001` 部分
                  binlog-pos: 15847                  # 对应 checkpoint 信息中的 `binlog_pos`
            
              - source-id: "mysql-replica-02"
                meta:
                  binlog-name: "mysql-bin.000456"
                  binlog-pos: 10485
            ```
            
            > **注意：**
            >
            > 如在 source 配置中启动了 `enable-gtid`，当前需要通过解析 binlog 或 relay log 文件获取 binlog position 对应的 GTID sets 并在 `meta` 中设置为 `binlog-gtid`。

4. 使用 [`start-task`](create-task.md) 命令以 v2.0.x 的数据迁移任务配置文件启动升级后的数据迁移任务。

5. 使用 [`query-status`](query-status.md) 命令确认数据迁移任务是否运行正常。
