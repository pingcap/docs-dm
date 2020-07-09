---
title: 创建数据同步任务
summary: 了解在部署 DM 集群后，如何快速创建数据同步任务。
category: how-to
---

# 创建数据同步任务

本文档介绍在 DM 集群部署成功后，如何快速创建简单的数据同步任务。

## 使用样例

在本地部署两个开启 binlog 的 MySQL 实例和一个 mocktkv 模式的 TiDB；通用 DM 集群的一个 DM-master 来管理集群和数据同步任务。各个节点的信息如下：

| 实例        | 服务器地址   | 端口   |
| :---------- | :----------- | :--- |
| MySQL1     | 127.0.0.1 | 3306 |
| MySQL2     | 127.0.0.1 | 3307 |
| TiDB       | 127.0.0.1 | 4000 |
| DM-master  | 127.0.0.1 | 8261 |

下面以此为例，说明如何创建数据同步任务。

### 运行上游 MySQL

运行两个 MySQL 服务，使用 Docker 启动 MySQL，示例命令如下：

{{< copyable "shell-regular" >}}

```bash
docker run --rm --name mysql-3306 -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3306 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3306.log 2>&1 &
docker run --rm --name mysql-3307 -p 3307:3307 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3307 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3307.log 2>&1 &
```

### 准备数据

向 mysql-3306 写入[示例数据](https://github.com/pingcap/dm/blob/bc1094a6b7388ad934279898b4e308cd3d58f7a9/tests/sharding/data/db1.prepare.sql)。

向 mysql-3307 写入[示例数据](https://github.com/pingcap/dm/blob/bc1094a6b7388ad934279898b4e308cd3d58f7a9/tests/sharding/data/db2.prepare.sql)。

### 运行下游 TiDB

使用以下命令运行一个 mocktikv 模式的 TiDB server：

{{< copyable "shell-regular" >}}

```bash
wget https://download.pingcap.org/tidb-v4.0.0-rc.2-linux-amd64.tar.gz
tar -xzvf tidb-v4.0.0-rc.2-linux-amd64.tar.gz
mv tidb-v4.0.0-rc.2-linux-amd64/bin/tidb-server ./
./tidb-server -P 4000 --store mocktikv --log-file "./tidb.log" &
```

## 配置 MySQL 数据源

运行数据同步任务前，需要对 source 进行配置，也就是 MySQL 的相关设置。

### 对密码进行加密

为了安全，建议配置及使用加密后的密码。使用 dmctl 对 MySQL/TiDB 的密码进行加密，以密码为 "123456" 为例：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

记录该加密后的密码，用于下面新建 MySQL 数据源。

> **注意：**
>
> 如果数据库没有设置密码，这可以跳过该步骤。

### 编写 source 配置文件

把以下配置文件内容写入到 `conf/source1.toml` 中。

MySQL1 的配置文件：

```toml
# MySQL1 Configuration.
 
source-id = "mysql-replica-01"

# 是否开启 GTID
enable-gtid = false
 
[from]
host = "127.0.0.1"
user = "root"
password = "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
port = 3306
```

对于 MySQL2 数据源，将以上内容复制到文件 `conf/source2.toml` 中，将 `conf/source2.toml` 配置文件中的 `name` 修改为 `mysql-replica-02`，并将 `password` 和 `port` 改为相应的值。

### 创建 source

在终端中执行下面的命令，使用 dmctl 将 MySQL1 的数据源配置加载到 DM 集群中：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --master-addr=127.0.0.1:8261 operate-source create conf/source1.toml
```

对于 MySQL2，将上面命令中的配置文件替换成 MySQL2 对应的配置文件。

## 创建数据同步任务

在导入[准备数据](#准备数据)后，MySQL1 和 MySQL2 实例中有若干个分表，这些分表的结构相同，所在库的名称都以 "sharding" 开头，表名称都以 "t" 开头，并且主键或唯一键不存在冲突（即每张分表的主键或唯一键各不相同）。现在需要把这些分表同步到 TiDB 中的 `db_target.t_target` 表中。

首先创建任务的配置文件：

{{< copyable "" >}}

```yaml
---
name: test
task-mode: all
is-sharding: true

target-database:
  host: "127.0.0.1"
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
./bin/dmctl -master-addr 127.0.0.1:8261 start-task conf/task.yaml
```

结果如下：

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

## 数据校验

修改上游 MySQL 分表中的数据，然后使用 [sync-diff-inspector](https://pingcap.com/docs-cn/stable/sync-diff-inspector/shard-diff/) 校验上下游数据是否一致，如果一致则说明同步任务运行正常。
