---
title: DM 集群性能测试
summary: 介绍如何测试 DM 集群的性能 
category: reference
---

# DM 集群性能测试

本文档介绍如何构建测试场景对 DM 集群进行性能测试，包括数据同步速度、延迟等。

## 同步数据流

可以使用简单的同步数据流来测试 DM 集群的数据同步性能，即单个 MySQL 实例到 TiDB 的数据同步：MySQL -> DM -> TiDB。

## 部署测试环境

- 使用 TiUP 部署 TiDB 测试集群，所有配置使用 TiUP 提供的默认配置。
- 部署 MySQL 服务，开启 row 模式 binlog，其他配置项使用默认配置。
- 部署 DM 集群，部署一个 DM-worker 和 一个 DM-master 即可。

## 性能测试

### 同步数据表结构

使用如下结构的表进行性能测试：

{{< copyable "sql" >}}

```sql
CREATE TABLE `sbtest` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `k` int(11) NOT NULL DEFAULT '0',
  `c` char(120) CHARSET utf8mb4 COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `pad` char(60) CHARSET utf8mb4 COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `k_1` (`k`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

### 全量导入性能测试用例

#### 生成测试数据

使用 `sysbench` 在上游创建测试表，并生成全量导入的测试数据。`sysbench` 生成数据的命令如下所示：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --mysql-host=172.16.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --table-size=50000000 prepare
```

#### 创建数据同步任务

创建上游 MySQL 的 source, source-id 配置为 `source-1`。

然后创建 `full` 模式的 DM 同步任务，示例任务配置文件如下：

```yaml
---
name: test-full
task-mode: full
is-sharding: false

# 使用实际测试环境中 TiDB 的信息配置
target-database:
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: ""

mysql-instances:
  -
    source-id: "source-1"
    black-white-list:  "instance"
    mydumper-config-name: "global"
    loader-thread: 16

# 配置 sysbench 生成数据所在的库的名称
black-white-list:
  instance:
    do-dbs: ["sbtest"]

mydumpers:
  global:
    rows: 32000
    threads: 32
```

> **注意：**
>
> - `mydumpers` 配置项的 `rows` 选项会开启单表多线程并发导出，加快数据导出速度。
> - `mysql-instances` 配置中的 `loader-thread` 以及 `mydumpers` 配置项中的 `rows` 和 `threads` 可以做适当调整，测试在不同配置下对性能的影响。

#### 获取测试结果

观察 DM-worker 日志，当出现 `all data files have been finished` 时表示全量数据导入完成，可以看到消耗时间。示例日志如下：

```
 [INFO] [loader.go:604] ["all data files have been finished"] [task=test] [unit=load] ["cost time"=52.439796ms]
```

根据测试数据的数据量和导入消耗时间，可以算出全量数据的同步速度。

## 增量同步性能测试用例

### 初始化表 

使用 `sysbench` 在上游创建测试表。

### 创建数据同步任务

创建上游 MySQL 的 source, source-id 配置为 `source-1`。（如果在全量同步性能测试中已经创建，则不需要再次创建）。

然后创建 `all` 模式的 DM 同步任务，示例任务配置文件如下：

```yaml
---
name: test-all
task-mode: all
is-sharding: false
enable-heartbeat: true

# 使用实际测试环境中 TiDB 的信息配置
target-database:
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: ""

mysql-instances:
  -
    source-id: "source-1"
    black-white-list:  "instance"
    syncer-config-name: "global"

# 配置 sysbench 生成数据所在的库的名称
black-white-list:
  instance:
    do-dbs: ["sbtest"]

syncers:
  global:
    worker-count: 16
    batch: 100
```

> **注意：**
>
> - `syncers` 配置项中的 `worker-count` 和 `batch` 可以做适当调整，测试在不同配置下对性能的影响。

### 生成增量数据

使用 `sysbench` 在上游持续生成增量数据，上游 `sysbench` 生成增量数据命令如下：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --num-threads=32 --mysql-host=172.17.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --report-interval=10 --time=1800 run
```

> **注意：**
>
> 可以调整 sysbench 的语句类型来测试在不同业务场景下 DM 的数据同步性能。

### 获取测试结果

通过 `query-status` 命令观测 DM 的同步状态，通过 Grafana 观测 DM 的监控指标。主要包括同步延迟 `replicate lag`，单位时间内完成的 job 数量 `finished sqls jobs` 等。
