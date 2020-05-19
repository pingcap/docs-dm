---
title: DM 集群性能测试
summary: 介绍如何测试 DM 集群的性能 
category: reference
---

# DM 集群性能测试

本文档介绍如何对 DM 集群进行性能测试，包括数据同步速度、延迟等。

## 测试场景

本节主要介绍如何构建测试场景。

### 同步数据流

因为目的是测试同步性能，可以使用简单的同步数据流，即单个 MySQL 实例到 TiDB 的数据同步：MySQL -> DM -> TiDB

### 公共配置信息

#### 同步数据表结构

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

#### 数据库配置

使用 TiDB Ansible 部署 TiDB 测试集群，所有配置使用 TiDB Ansible 提供的默认配置。

### 全量导入性能测试用例

#### 测试过程

- 部署测试环境
- 使用 `sysbench` 在上游创建测试表，并生成全量导入的测试数据
- 在 `full` 模式下启动 DM 同步任务

`sysbench` 生成数据的命令如下所示：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --mysql-host=172.16.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --table-size=50000000 prepare
```

#### 全量导入性能测试

在任务配置文件中的 `mydumpers` 配置项中配置 `rows` 选项开启单表多线程并发导出（当前 `mydumper` 在 MySQL 的单表并发会优先选出一列做拆分基准，选择优先级为主键>唯一索引>普通索引，选出目标列后需保证该列为 INT 类型；针对 `TiDB` 的并发导出则会优先尝试 `_tidb_rowid` 列），以下的第一项测试用于验证开启单表并发导出可以提高数据导出性能。

| 测试项             | 导出线程数  | mydumper extra-args 参数            | 导出速度 (MB/s)   |
| :----------------: | :---------: | :---------------------------------: | :---------------: |
| 开启单表并发导出   | 32          | "--rows 320000 --regex '^sbtest.*'" | 191.03            |
| 不开启单表并发导出 |  4          | "--regex '^sbtest.*'"               | 72.22             |

| 测试项    | 事务执行延迟 (s) | 每条插入语句包含的行数 | 导入数据量 (GB) | 导入时间 (s) | 导入速度 (MB/s) |
| :-------: | :--------------: | :--------------------: | :-------------: | :----------: | :-------------: |
| load data | 1.737            | 4878                   | 38.14           | 2346.9       | 16.64           |

#### 在 load 处理单元使用不同 pool size 的性能测试对比

该测试中全量导入的数据量为 3.78 GB，使用以下 `sysbench` 命令生成：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --mysql-host=172.16.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --table-size=5000000 prepare
```

| load 处理单元 pool size | 事务执行时间 (s) | 导入时间 (s) | 导入速度 (MB/s) | TiDB 99 duration (s) |
| :---------------------: | :--------------: | :----------: | :-------------: | :------------------: |
| 2                       | 0.250            | 425.9        | 9.1             | 0.23                 |
| 4                       | 0.523            | 360.1        | 10.7            | 0.41                 |
| 8                       | 0.986            | 267.0        | 14.5            | 0.93                 |
| 16                      | 2.022            | 265.9        | 14.5            | 2.68                 |
| 32                      | 3.778            | 262.3        | 14.7            | 6.39                 |
| 64                      | 7.452            | 281.9        | 13.7            | 8.00                 |

#### 导入数据时每条插入语句包含行数不同的情况下的性能测试对比

该测试中全量导入的数据量为 3.78 GB，load 处理单元 `pool-size` 大小为 32。插入语句包含行数通过 mydumper 的 `--statement-size` 来控制。

| 每条语句中包含的行数       | mydumper extra-args 参数  | 事务执行时间 (s)  | 导入时间 (s) | 导入速度 (MB/s) | TiDB 99 duration (s) |
| :------------------------: | :-----------------------: | :---------------: | :----------: | :-------------: | :------------------: |
|            7426            | -s 1500000 -r 320000      |   6.982           |  258.3       |     15.0        |        10.34         |
|            4903            | -s 1000000 -r 320000      |   3.778           |  262.3       |     14.7        |         6.39         |
|            2470            | -s 500000 -r 320000       |   1.962           |  271.36      |     14.3        |         2.00         |
|            1236            | -s 250000 -r 320000       |   1.911           |  283.3       |     13.7        |         1.50         |
|            618             | -s 125000 -r 320000       |   0.683           |  299.9       |     12.9        |         0.73         |
|            310             |  -s 62500 -r 320000       |   0.413           |  322.6       |     12.0        |         0.49         |

### 增量同步性能测试用例

#### 测试过程

- 部署测试环境
- 使用 `sysbench` 在上游创建测试表，并生成全量导入的测试数据
- 在 `all` 模式下启动 DM 同步任务，等待同步任务进入 `sync` 同步阶段
- 使用 `sysbench` 在上游持续生成增量数据，通过 `query-status` 命令观测 DM 的同步状态，通过 Grafana 观测 DM 和 TiDB 的监控指标。

#### 增量同步性能测试结果

上游 `sysbench` 生成增量数据命令

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --num-threads=32 --mysql-host=172.17.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --report-interval=10 --time=1800 run
```

该性能测试中同步任务 `sync` 处理单元 `worker-count` 设置为 32，`batch` 大小设置为 100。


#### 在 sync 处理单元使用不同并发度的性能测试对比


#### 不同数据分布的增量同步性能测试对比

| sysbench 语句类型| DM relay log 持久化速率 (MB/s) | DM 内部 job tps | DM 事务执行时间 (ms) | TiDB qps | TiDB 99 duration (ms) |
| :--------------: | :----------------------------: | :-------------: | :------------------: | :------: | :-------------------: |
| insert_only      | 11.3                           | 23345           | 28                   | 29.2k    | 10                    |
| write_only       | 18.7                           | 33470           | 129                  | 34.6k    | 11                    |

## 推荐同步任务参数配置

### dump 处理单元

推荐每一条插入语句的大小在 200KB ~ 1MB 之间，相应每条语句包含的行数大约在 1000-5000（具体包含的语句行数与实际场景中每行数据大小有关）。

### load 处理单元

推荐 `pool-size` 设置为 16。

### sync 处理单元

推荐将 `batch` 设置为 100，`worker-count` 设置为 16 ~ 32。