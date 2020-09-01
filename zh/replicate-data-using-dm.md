---
title: 使用 DM 迁移数据
aliases: ['/docs-cn/tidb-data-migration/stable/replicate-data-using-dm/','/docs-cn/tidb-data-migration/v1.0/replicate-data-using-dm/','/docs-cn/dev/reference/tools/data-migration/deploy/','/docs-cn/v3.1/reference/tools/data-migration/deploy/','/docs-cn/v3.0/reference/tools/data-migration/deploy/','/docs-cn/v2.1/reference/tools/data-migration/deploy/']
---

# 使用 DM 迁移数据

本文介绍如何使用 DM (Data Migration) 迁移数据。

## 第 1 步：部署 DM 集群

目前推荐使用 DM-Ansible 部署 DM 集群，具体部署方法参照 [使用 DM-Ansible 部署 DM 集群](deploy-a-dm-cluster-using-ansible.md)；也可以使用 binary 部署 DM 集群用于体验或者测试，具体部署方法参照[使用 DM binary 部署 DM 集群](deploy-a-dm-cluster-using-binary.md)。

> **注意：**
>
> - 在 DM 所有的配置文件中，数据库的密码要使用 dmctl 加密后的密文。如果数据库密码为空，则不需要加密。关于如何使用 dmctl 加密明文密码，参考[使用 dmctl 加密上游 MySQL 用户密码](deploy-a-dm-cluster-using-ansible.md#使用-dmctl-加密上游-mysql-用户密码)。
> - 上下游数据库用户必须拥有相应的读写权限。

## 第 2 步：检查集群信息

使用 DM-Ansible 部署 DM 集群后，相关配置信息如下：

- DM 集群相关组件配置信息

    | 组件 | 主机 | 端口 |
    |:------|:---- |:---- |
    | dm_worker1 | 172.16.10.72 | 8262 |
    | dm_worker2 | 172.16.10.73 | 8262 |
    | dm_master | 172.16.10.71 | 8261 |

- 上下游数据库实例相关信息

    | 数据库实例 | 主机 | 端口 | 用户名 | 加密密码 |
    |:-------- |:--- | :--- | :--- | :--- |
    | 上游 MySQL-1 | 172.16.10.81 | 3306 | root | VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU= |
    | 上游 MySQL-2 | 172.16.10.82 | 3306 | root | VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU= |
    | 下游 TiDB | 172.16.10.83 | 4000 | root | |

- dm-master 进程配置文件 `{ansible deploy}/conf/dm-master.toml` 中的配置

    ```toml
    # Master 配置
    
    # DM-worker 是否使用全局事务标识符 (GTID) 拉取 binlog。使用前提是在上游 MySQL 已开启 GTID 模式。
    enable-gtid = false

    [[deploy]]
    source-id = "mysql-replica-01"
    dm-worker = "172.16.10.72:8262"

    [[deploy]]
    source-id = "mysql-replica-02"
    dm-worker = "172.16.10.73:8262"
    ```

    > **注意：**
    >
    > `{ansible deploy}/conf/dm-master.toml` 中的 `{ansible deploy}` 表示使用 DM-Ansible 部署 DM 时通过 `deploy_dir` 参数指定的目录。

## 第 3 步：配置任务

假设需要将 MySQL-1 和 MySQL-2 实例的 `test_db` 库的 `test_table` 表以**全量+增量**的模式迁移到下游 TiDB 的 `test_db` 库的 `test_table` 表。

复制并编辑 `{ansible deploy}/conf/task.yaml.example`，生成如下任务配置文件 `task.yaml`：

```yaml
# 任务名，多个同时运行的任务不能重名。
name: "test"
# 全量+增量 (all) 迁移模式。
task-mode: "all"
# 下游 TiDB 配置信息。
target-database:
  host: "172.16.10.83"
  port: 4000
  user: "root"
  password: ""

# 当前数据迁移任务需要的全部上游 MySQL 实例配置。
mysql-instances:
-
  # 上游实例或者复制组 ID，参考 `inventory.ini` 的 `source_id` 或者 `dm-master.toml` 的 `source-id 配置`。
  source-id: "mysql-replica-01"
  # 需要迁移的库名或表名的黑白名单的配置项名称，用于引用全局的黑白名单配置，全局配置见下面的 `block-allow-list` 的配置。
  block-allow-list: "global"          # 如果 DM 版本 <= v1.0.6 则使用 black-white-list。
  # Mydumper 的配置项名称，用于引用全局的 Mydumper 配置。
  mydumper-config-name: "global"

-
  source-id: "mysql-replica-02"
  block-allow-list: "global"          # 如果 DM 版本 <= v1.0.6 则使用 black-white-list。
  mydumper-config-name: "global"

# 黑白名单全局配置，各实例通过配置项名引用。
block-allow-list:                     # 如果 DM 版本 <= v1.0.6 则使用 black-white-list。
  global:
    do-tables:                        # 需要迁移的上游表的白名单。
    - db-name: "test_db"              # 需要迁移的表的库名。
      tbl-name: "test_table"          # 需要迁移的表的名称。

# Mydumper 全局配置，各实例通过配置项名引用。
mydumpers:
  global:
    mydumper-path: "./bin/mydumper"   # Mydumper 二进制文件的路径。
    extra-args: "-B test_db -T test_table"  # mydumper 的其他参数，从 DM 1.0.2 版本开始，DM 会自动生成 table-list 配置，在其之前的版本仍然需要人工配置。
```

## 第 4 步：启动任务

为了提前发现数据迁移任务的一些配置错误，DM 中增加了[前置检查](precheck.md)功能：

- 启动数据迁移任务时，DM 自动检查相应的权限和配置。
- 也可使用 `check-task` 命令手动前置检查上游的 MySQL 实例配置是否符合 DM 的配置要求。

> **注意：**
>
> 第一次启动数据迁移任务时，必须确保上游数据库已配置。否则，启动任务时会报错。

1. 进入 dmctl 目录 `/home/tidb/dm-ansible/resources/bin/`。

2. 执行以下命令启动 dmctl。

    {{< copyable "shell-regular" >}}

    ```bash
    ./dmctl --master-addr 172.16.10.71:8261
    ```

3. 执行以下命令启动数据迁移任务。其中，`task.yaml` 是之前编辑的配置文件。

    {{< copyable "" >}}

    ```bash
    » start-task ./task.yaml
    ```

    - 如果执行该命令后返回的结果如下，则表明任务已成功启动。

        ```json
        {
            "result": true,
            "msg": "",
            "workers": [
                {
                    "result": true,
                    "worker": "172.16.10.72:8262",
                    "msg": ""
                },
                {
                    "result": true,
                    "worker": "172.16.10.73:8262",
                    "msg": ""
                }
            ]
        }
        ```

    - 如果任务启动失败，可根据返回结果的提示进行配置变更后执行 `start-task task.yaml` 命令重新启动任务。

## 第 5 步：查询任务

如需了解 DM 集群中是否存在正在运行的迁移任务及任务状态等信息，可在 dmctl 内使用以下命令进行查询：

{{< copyable "" >}}

```bash
» query-status
```

## 第 6 步：停止任务

如果不再需要进行数据迁移，可以在 dmctl 内使用以下命令停止迁移任务：

{{< copyable "" >}}

```bash
» stop-task test
```

其中的 `test` 是 `task.yaml` 配置文件中 `name` 配置项设置的任务名。

## 第 7 步：监控任务与查看日志

如果使用 DM-Ansible 部署 DM 集群时，正确部署了 Prometheus、Alertmanager 与 Grafana，且其地址均为 `172.16.10.71`。可在浏览器中打开 <http://172.16.10.71:9093> 进入 Alertmanager 查看 DM 告警信息；可在浏览器中打开 <http://172.16.10.71:3000> 进入 Grafana，选择 DM 的 dashboard 查看 DM 相关监控项。

DM 在运行过程中，DM-worker, DM-master 及 dmctl 都会通过日志输出相关信息。各组件的日志目录如下：

- DM-master 日志目录：通过 DM-master 进程参数 `--log-file` 设置。如果使用 DM-Ansible 部署 DM，则日志目录位于 DM-master 节点的 `{ansible deploy}/log/dm-master.log`。
- DM-worker 日志目录：通过 DM-worker 进程参数 `--log-file` 设置。如果使用 DM-Ansible 部署 DM，则日志目录位于 DM-worker 节点的 `{ansible deploy}/log/dm-worker.log`。
- dmctl 日志目录：与其二进制文件目录相同。
