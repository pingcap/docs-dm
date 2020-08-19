---
title: 从 MySQL 迁移数据 —— 以 Amazon Aurora MySQL 为例
summary: 使用 DM 从 MySQL/Amazon Aurora MySQL 迁移数据。
aliases: ['/docs-cn/tidb-data-migration/dev/migrate-from-mysql-aurora/']
---

# 从 MySQL 迁移数据 —— 以 Amazon Aurora MySQL 为例

本文以 [Amazon Aurora MySQL](https://aws.amazon.com/cn/rds/aurora/details/mysql-details/) 为例介绍如何使用 DM 从 MySQL 兼容的数据库迁移数据到 TiDB。

示例使用的 Aurora 集群信息如下：

| 集群 | 终端节点 | 端口 | 角色 | 版本 |
|:-------- |:--- | :--- | :--- |:---|
| Aurora-1 | test-dm-2-0.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | 写入器 | Aurora (MySQL)-5.7.12 |
| Aurora-1 | test-dm-2-0.cluster-ro-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | 读取器 | Aurora (MySQL)-5.7.12 |
| Aurora-2 | test-dm-2-0-2.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | 写入器 | Aurora (MySQL)-5.7.12 |
| Aurora-2 | test-dm-2-0-2.cluster-ro-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | 读取器 | Aurora (MySQL)-5.7.12 |

Aurora 集群中数据与迁移计划如下：

| 集群 | 数据库 | 表 | 是否迁移 |
|:---- |:---- | :--- | :--- |
| Aurora-1 | migrate_me | t1 | 是 |
| Aurora-1 | ignore_me | ignore_table | 否 |
| Aurora-2 | migrate_me | t2 | 是 |
| Aurora-2 | ignore_me | ignore_table | 否 |

迁移使用的 Aurora 集群用户如下

| 集群 | 用户 | 密码 |
|:---- |:---- | :--- |
| Aurora-1 | root | 12345678 |
| Aurora-2 | root | 12345678 |

示例使用的 TiDB 集群信息如下。该集群使用 [Cloud TiDB](https://tidbcloud.com/) 服务一键部署：

| 节点 | 端口 | 版本 |
|:--- | :--- | :--- |
| tidb.6657c286.23110bc6.us-east-1.prod.aws.tidbcloud.com | 4000 | v4.0.2 |

迁移使用的 TiDB 集群用户如下

| 用户 | 密码 |
|:---- | :--- |
| root | 87654321 |

预期迁移后，TiDB 集群中存在``` `migrate_me`.`t1` ```与``` `migrate_me`.`t2` ```，其中数据与 Aurora 集群一致。

> **注意：**
>
> 本次迁移不涉及合库合表，如需使用合库合表请参考 [DM 合库合表场景](https://docs.pingcap.com/zh/tidb-data-migration/dev/scenarios#%E5%90%88%E5%BA%93%E5%90%88%E8%A1%A8%E5%9C%BA%E6%99%AF) 。

## 第 1 步：数据迁移前置条件

### DM 部署节点

DM 作为数据迁移的核心，需要正常连接上游 Aurora 与下游 TiDB 集群，因此通过 MySQL client 等方式检查部署 DM 的节点能连通上下游。除此以外，DM 对软硬件要求可以参见 [DM 集群软硬件环境需求](hardware-and-software-requirements.md)。

### Aurora

DM 在增量同步阶段依赖 `ROW` 格式的 binlog，请按照 [AWS 官网流程](https://aws.amazon.com/cn/premiumsupport/knowledge-center/enable-binary-logging-aurora/)进行配置。

CHECK
> **注意：**
>
> Aurora 读取器不能开启 binlog，因此不能作为 DM 数据迁移时的上游 master server。

如果需要基于 GTID 进行数据迁移，需要将 `gtid-mode` 与 `enforce_gtid_consistency` 均设置为 `ON`，请按照[为 Aurora 集群启用 GTID 支持](https://docs.aws.amazon.com/zh_cn/AmazonRDS/latest/AuroraUserGuide/mysql-replication-gtid.html#mysql-replication-gtid.configuring-aurora)进行配置。

> **注意：**
>
> 基于 GTID 的数据迁移需要 MySQL 5.7 (Aurora 2.04) 或更高版本。

除这些 Aurora 特有配置以外，上游数据库需满足权限、表结构等迁移 MySQL 的其他要求，参见[上游 MySQL 实例检查内容](precheck.md#检查内容)。

## 第 2 步：部署 DM 集群

DM 可以通过多种方式进行部署，目前推荐使用 TiUP 部署 DM 集群，具体部署方法参照[使用 TiUP 部署 DM 集群](deploy-a-dm-cluster-using-tiup.md)。示例有两个数据源，因此需要至少部署两个 DM-worker 节点。

部署完成后，需要记录任意一台 DM-master 节点的 IP 和服务端口（默认为 `8261`），以供 `dmctl` 连接。本示例使用 `127.0.0.1:8261`。通过 TiUP 使用 `dmctl` 检查 DM 状态：

{{< copyable "shell-regular" >}}

```bash
./tiup dmctl --master-addr 127.0.0.1:8261 query-status
```

返回值为

```bash
{
    "result": true,
    "msg": "",
    "tasks": []
}
```

## 第 3 步：配置数据源

> **注意：**
>
> - DM 所使用的配置文件支持明文数据库密码或密文数据库密码。关于如何获得密文数据库密码，参考[使用 dmctl 加密上游 MySQL 用户密码](deploy-a-dm-cluster-using-ansible.md#使用-dmctl-加密上游-mysql-用户密码)。

根据示例信息保存如下的数据源配置文件：

文件 `source1.yaml`

```yaml
# Aurora-1
source-id: "aurora-replica-01"

enable-gtid: false

from:
  host: "test-dm-2-0.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com"
  user: "root"
  password: "12345678"
  port: 3306
```

文件 `source2.yaml`

```yaml
# Aurora-2
source-id: "aurora-replica-02"

enable-gtid: false

from:
  host: "test-dm-2-0-2.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com"
  user: "root"
  password: "12345678"
  port: 3306
```

参照[使用 DM 同步数据：创建数据源](https://docs.pingcap.com/zh/tidb-data-migration/dev/replicate-data-using-dm#%E7%AC%AC-3-%E6%AD%A5%E5%88%9B%E5%BB%BA%E6%95%B0%E6%8D%AE%E6%BA%90)，通过 TiUP 使用 `dmctl` 添加两个数据源。

{{< copyable "shell-regular" >}}

```bash
./tiup dmctl --master-addr 127.0.0.1:8261 operate-source create dm-test/source1.yaml
./tiup dmctl --master-addr 127.0.0.1:8261 operate-source create dm-test/source2.yaml
```

每个数据源的返回信息中，应当包含了一个与之绑定的 DM-worker。

```bash
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "aurora-replica-01",
            "worker": "one-dm-worker-ID"
        }
    ]
}
```

## 第 4 步：配置任务

本示例选择同步 Aurora 已有数据并将新增数据实时同步给 TiDB，即**全量+增量**模式。根据上文中的 TiDB 集群信息、添加数据源 ID、要同步以及忽略的表，保存如下任务配置文件 `task.yaml`：

```yaml
# 任务名，多个同时运行的任务不能重名。
name: "test"
# 全量+增量 (all) 同步模式。
task-mode: "all"
# 下游 TiDB 配置信息。
target-database:
  host: "tidb.6657c286.23110bc6.us-east-1.prod.aws.tidbcloud.com"
  port: 4000
  user: "root"
  password: "87654321"

# 当前数据同步任务需要的全部上游 MySQL 实例配置。
mysql-instances:
- source-id: "aurora-replica-01"
  # 需要同步的库名或表名的黑白名单的配置项名称，用于引用全局的黑白名单配置，全局配置见下面的 `block-allow-list` 的配置。
  block-allow-list: "global"

- source-id: "aurora-replica-02"
  block-allow-list: "global"

# 黑白名单配置。
block-allow-list:
  global:                             # 被上文 block-allow-list: "global" 所引用
    do-dbs: ["migrate_me"]            # 需要同步的上游数据库白名单。
    ignore-dbs: ["ignore_me"]         # 需要同步的表的库名。
```

## 第 5 步：启动任务

通过 TiUP 使用 `dmctl` 启动任务。

{{< copyable "shell-regular" >}}

```bash
./tiup dmctl --master-addr 127.0.0.1:8261 start-task task.yaml
```

启动成功时的返回信息是

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "aurora-replica-01",
            "worker": "one-dm-worker-ID"
        },
        {
            "result": true,
            "msg": "",
            "source": "aurora-replica-02",
            "worker": "another-dm-worker-ID"
        }
    ]
}
```

## 第 6 步：查询任务

如需了解 DM 集群中是否存在正在运行的同步任务及任务状态等信息，可在 dmctl 内使用以下命令进行查询：

{{< copyable "shell-regular" >}}

```bash
query-status
```

> **注意：**
>
> 如果查询命令的返回结果中包含以下错误信息，则表明在全量同步的 dump 阶段不能获得相应的 lock：
>
> ```
> Couldn't acquire global lock, snapshots will not be consistent: Access denied for user 'root'@'%' (using password: YES)
> ```
>
> 此时如果能接受不使用 FTWL 来确保 dump 文件与 metadata 的一致或上游能暂时停止写入，可以通过为 `mydumpers` 下的 `extra-args` 添加 `--no-locks` 参数来进行绕过，具体方法为：
>
> 1. 使用 `stop-task` 停止当前由于不能正常 dump 而已经转为 paused 的任务
> 2. 将原 `task.yaml` 中的 `extra-args: "-B test_db -T test_table"` 更新为 `extra-args: "-B test_db -T test_table --no-locks"`
> 3. 使用 `start-task` 重新启动任务
