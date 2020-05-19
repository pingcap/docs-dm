---
title: 创建数据同步任务及测试验证
summary: 部署 DM 集群后，创建数据同步任务验证集群是否可以正常工作。
category: reference
---

# 测试验证

在 DM 集群部署成功后，创建简单的数据同步任务，验证集群是否可以正常工作。

## 配置 MySQL 数据源

运行数据同步任务前，需要对 source 进行配置，也就是 MySQL 的相关设置。且为了安全，建议用户配置及使用加密后的密码。首先使用 dmctl 对 MySQL 的密码进行加密，以密码为 "123456" 为例：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

记录该加密后的密码，用于下面 MySQL1 的配置。把以下配置文件内容写入到 `conf/source1.toml` 中。

MySQL1 的配置文件：

```toml
# MySQL1 Configuration.
 
source-id = "mysql-replica-01"

# 是否开启 GTID
enable-gtid = false
 
[from]
host = "192.168.0.1"
user = "root"
password = "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
port = 3306
```

在终端中执行下面的命令，使用 dmctl 将数据源配置加载到 DM 集群中：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --master-addr=192.168.0.4:8261 operate-source create conf/source1.toml
```

对于 MySQL2，修改配置文件中的 `name` 为 `mysql-replica-02`，`host` 为 `192.168.0.2`，并将 `password` 和 `port` 改为相应的值即可。

## 创建数据同步任务

假设在 MySQL1 和 MySQL2 实例中有若干个分表，这些分表的结构相同，所在库的名称都以 "sharding" 开头，表名称都以 "t" 开头，并且主键或唯一键不存在冲突（即每张分表的主键或唯一键各不相同）。现在需要把这些分表同步到 TiDB 中的 `db_target.t_target` 表中。

首先创建任务的配置文件：

{{< copyable "" >}}

```yaml
---
name: test
task-mode: all
is-sharding: true

target-database:
  host: "192.168.0.3"
  port: 4000
  user: "root"
  password: "" # 如果密码不为空，也需要配置 dmctl 加密后的密码

mysql-instances:
  - source-id: "mysql-replica-01"
    black-white-list:  "instance"
    route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
    mydumper-thread: 4
    loader-thread: 16
    syncer-thread: 16

  - source-id: "mysql-replica-02"
    black-white-list:  "instance"
    route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
    mydumper-thread: 4
    loader-thread: 16
    syncer-thread: 16

black-white-list:
  instance:
    do-dbs: ["~^sharding[\\d]+"]
    do-tables:
    -  db-name: "~^sharding[\\d]+"
       tbl-name: "~^t[\\d]+"

routes:
  sharding-route-rules-table:
    schema-pattern: sharding*
    table-pattern: t*
    target-schema: db_target
    target-table: t_target

  sharding-route-rules-schema:
    schema-pattern: sharding*
    target-schema: db_target
```

将以上配置内容写入到 `conf/task.yaml` 文件中，使用 dmctl 创建任务：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl -master-addr 192.168.0.4:8261
```

```
Welcome to dmctl
Release Version: v1.0.0-69-g5134ad1
Git Commit Hash: 5134ad19fbf6c57da0c7af548f5ca2a890bddbe4
Git Branch: master
UTC Build Time: 2019-04-29 09:36:42
Go Version: go version go1.12 linux/amd64
»
```

{{< copyable "" >}}

```bash
» start-task conf/task.yaml
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        },
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-02",
            "worker": "worker2"
        }
    ]
}
```

这样就成功创建了一个将 MySQL1 和 MySQL2 实例中的分表数据同步到 TiDB 的任务。

## 测试验证

修改上游 MySQL 分表中的数据，然后使用 [sync-diff-inspector](https://pingcap.com/docs-cn/stable/sync-diff-inspector/shard-diff/) 校验上下游数据是否一致，如果一致则说明同步任务运行正常。