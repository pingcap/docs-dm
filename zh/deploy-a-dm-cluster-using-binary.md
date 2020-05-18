---
title: 使用 DM binary 部署 DM 集群
category: how-to
---

# 使用 DM binary 部署 DM 集群

本文将介绍如何使用 DM binary 快速部署 DM 集群。

## 准备工作

下载官方 binary，链接地址：[DM 下载](https://pingcap.com/docs-cn/stable/reference/tools/download/#tidb-dm-data-migration)。

下载的文件中包括子目录 bin 和 conf。bin 目录下包含 dm-master、dm-worker 以及 dmctl 的二进制文件。conf 目录下有相关的示例配置文件。

## 使用样例

假设在两台服务器上部署 MySQL，在一台服务器上部署 TiDB（mocktikv 模式），另外在五台服务器上部署两个 DM-worker 实例和三个 DM-master 实例。各个节点的信息如下：

| 实例        | 服务器地址   |
| :---------- | :----------- |
| MySQL1     | 192.168.0.1 |
| MySQL2     | 192.168.0.2 |
| TiDB       | 192.168.0.3 |
| DM-master1 | 192.168.0.4 |
| DM-master2 | 192.168.0.5 |
| DM-master3 | 192.168.0.6 |
| DM-worker1 | 192.168.0.7 |
| DM-worker2 | 192.168.0.8 |

MySQL1 和 MySQL2 中需要开启 binlog。下面以此为例，说明如何部署 DM。

### DM-master 的部署

DM-master 提供[命令行参数](#使用命令行参数部署-dm-master)和[配置文件](#使用配置文件部署-dm-master)两种配置方式。

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

# 初始集群中所有 DM-master 的 advertise-peer-urls 的值
initial-cluster = "master1=http://192.168.0.4:8291,master2=http://192.168.0.5:8291,master3=http://192.168.0.6:8291"
```

在终端中使用下面的命令运行 DM-master：

{{< copyable "shell-regular" >}}

```bash
./bin/dm-master -config conf/dm-master1.toml
```

对于 DM-master2 和 DM-master3，修改配置文件中的 `name` 为 `master2` 和 `master3`，并将 `peer-urls` 的值改为 `192.168.0.5:8291` 和 `192.168.0.6:8291` 即可。

### DM-worker 的部署

DM-worker 提供[命令行参数](#使用命令行参数部署-dm-worker)和[配置文件](#使用配置文件部署-dm-worker)两种配置方式。

#### 使用命令行参数部署 DM-worker

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

#### 使用配置文件部署 DM-worker

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
join = "192.168.0.4:8261,192.168.0.5:8261,192.168.0.6:8261"
```

在终端中使用下面的命令运行 DM-worker：

{{< copyable "shell-regular" >}}

```bash
./bin/dm-worker -config conf/dm-worker1.toml
```

对于 DM-worker2，修改配置文件中的 `name` 为 `worker2` 即可。

这样，DM 集群就部署成功了。
