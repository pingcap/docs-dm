---
title: 使用 DM binary 部署 DM 集群
category: how-to
---

# 使用 DM binary 部署 DM 集群

本文将介绍如何使用 DM binary 快速部署 DM 集群。

## 准备工作

下载官方 binary，链接地址：[DM 下载](https://pingcap.com/docs-cn/stable/reference/tools/download/#tidb-dm-data-migration)。

下载的文件中包括子目录 bin 和 conf。bin 目录下包含 dm-master、dm-worker、dmctl 以及 Mydumper 的二进制文件。conf 目录下有相关的示例配置文件。

## 使用样例

假设在两台服务器上部署 MySQL，在一台服务器上部署 TiDB（mocktikv 模式），另外在四台服务器上部署两个 DM-worker 实例和两个 DM-master 实例。各个节点的信息如下：

| 实例        | 服务器地址   |
| :---------- | :----------- |
| MySQL1     | 192.168.0.1 |
| MySQL2     | 192.168.0.2 |
| TiDB       | 192.168.0.3 |
| DM-master1 | 192.168.0.4 |
| DM-master2 | 192.168.0.5 |
| DM-worker1 | 192.168.0.6 |
| DM-worker2 | 192.168.0.7 |

MySQL1 和 MySQL2 中需要开启 binlog。下面以此为例，说明如何部署 DM。

### DM-master 的部署

DM-master 提供[命令行参数](#使用命令行参数部署-DM-master)和[配置文件](#使用配置文件部署-DM-master)两种配置方式。

#### 使用命令行参数部署 DM-master

DM-master 的命令行参数说明：

```bash
./bin/dm-master --help
```

```
Usage of dm-master:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${master-addr}")
  -advertise-peer-urls string
        advertise URLs for peer traffic (default "${peer-urls}")
  -config string
        path to config file
  -data-dir string
        path to the data directory (default "default.${name}")
  -initial-cluster string
        initial cluster configuration for bootstrapping, e.g. dm-master=http://127.0.0.1:8291
  -join string
        join to an existing cluster (usage: cluster's "${master-addr}" list, e.g. "127.0.0.1:8261,127.0.0.1:18261"
  -log-file string
        log file path
  -master-addr string
        master API server and status addr
  -name string
        human-readable name for this DM-master member
  -peer-urls string
        URLs for peer traffic (default "http://127.0.0.1:8291")
  -print-sample-config
        print sample config file of dm-worker
```

> **注意：**
>
> 某些情况下，无法使用命令行参数来配置 DM-master，因为有的配置并未暴露给命令行。

#### 使用配置文件部署 DM-master

推荐使用配置文件，把以下配置文件内容写入到 `conf/dm-master1.toml` 中。

DM-master 的配置文件：

```toml
# Master Configuration.

name = "master1"

# 日志配置
log-level = "info"
log-file = "dm-master.log"

# DM-master 监听地址
master-addr = ":8261"

# DM-master 节点的对等 URL
peer-urls = "192.168.0.4:8291"

# 初始集群中所有 DM-master 的 advertise-peer-urls 的值。
initial-cluster = "master1=http://192.168.0.4:8291,master2=http://192.168.0.5:8292"
```

在终端中使用下面的命令运行 DM-master：

{{< copyable "shell-regular" >}}

```bash
./bin/dm-master -config conf/dm-master1.toml
```

对于 DM-master2，修改配置文件中的 `name` 为 `master2`，并将 `master-addr` 的端口改为 `8361` ，将 `peer-urls` 的改为 `192.168.0.5:8292` 即可。

### DM-worker 的部署

DM-worker 提供命令行参数和配置文件两种配置方式。

**配置方式 1：命令行参数**

查看 DM-worker 的命令行参数说明：

{{< copyable "shell-regular" >}}

```bash
./bin/dm-worker --help
```

```
Usage of worker:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${worker-addr}")
  -config string
        path to config file
  -join string
        join to an existing cluster (usage: dm-master cluster's "${master-addr}")
  -keepalive-ttl int
        dm-worker's TTL for keepalive with etcd (in seconds) (default 10)
  -log-file string
        log file path
  -name string
        human-readable name for DM-worker member
  -print-sample-config
        print sample config file of dm-worker
  -worker-addr string
        listen address for client traffic
```

> **注意：**
>
> 某些情况下，无法使用命令行参数的方法来配置 DM-worker，因为有的配置并未暴露给命令行。

**配置方式 2：配置文件**

推荐使用配置文件来配置 DM-worker，把以下配置文件内容写入到 `conf/dm-worker1.toml` 中。

DM-worker 的配置文件：

```toml
# Worker Configuration.

name = "worker1"

# 日志配置
log-level = "info"
log-file = "dm-worker.log"

# DM-worker 的地址
worker-addr = ":8262"

# 对应集群中 DM-master 配置中的 master-addr
join = "192.168.0.4:8261,192.168.0.5:8361"
```

在终端中使用下面的命令运行 DM-worker：

{{< copyable "shell-regular" >}}

```bash
./bin/dm-worker -config conf/dm-worker1.toml
```

对于 DM-worker2，修改配置文件中的 `name` 为 `worker2`，并将 `worker-addr` 的端口改为 `8263` 即可。如果因为没有多余的机器，将 DM-worker2 与 DM-worker1 部署在一台机器上，需要把两个 DM-worker 实例部署在不同的路径下，否则保存元信息和 relay log 的默认路径会冲突。

### 配置 MySQL 数据源

运行数据同步任务前，需要对 source 进行配置，也就是 MySQL 的相关设置。且为了安全，强制用户配置加密后的密码。首先使用 dmctl 对 MySQL 的密码进行加密，以密码为 "123456" 为例：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

记录该加密后的值，用于下面 MySQL1 的配置。把以下配置文件内容写入到 `conf/source1.toml` 中。

MySQL1 的配置文件

如需要启用 GTID 模式，则额外配置 enable-gtid = true。

```toml
# MySQL1 Configuration.
 
source-id = "mysql-replica-01"
 
[from]
host = "192.168.0.1"
user = "root"
password = "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
port = 3306
```

在终端中使用下面的命令，使用 dmctl 将数据源配置加载到 DM 集群中 ：

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --master-addr=192.168.0.4:8261 operate-source create conf/source1.toml
```

对于 MySQL2 ，修改配置文件中的 `name` 为 `mysql-replica-02`， `host` 为 `192.168.0.2`，并将 `password` 和 `port` 改为相应的值即可。

这样，DM 集群就部署成功了。下面创建简单的数据同步任务来使用 DM 集群。

### 创建数据同步任务

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
