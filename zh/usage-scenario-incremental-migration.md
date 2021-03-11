---
title: Data Migration 增量数据迁移场景
aliases: ['/docs-cn/tidb-data-migration/dev/usage-scenario-incremental-migration/','/zh/tidb-data-migration/dev/usage-scenario-incremental-migration']
---

# Data Migration 增量数据迁移场景

本文介绍如何使用 DM 将源数据库从指定位置开始的 Binlog 同步到下游 TiDB。本文以一个上游 MySQL 实例进行迁移为例。

## 上游实例

假设上游实例为：

| Schema | Tables |
|:------|:------|
| user  | information, log |
| store | store_bj, store_tj |
| log   | messages |

## 迁移要求

将 `log` 库从某个时间点起的数据变动同步到 TiDB 集群。

> **注意：**
>
> - 如果同步起始位置会造成某些表从中途开始同步，需要在下游手动创建该表（如果使用了 [Table routine](key-features.md#table-routing) 功能需要填写迁移后的表名）。除此以外，同步时可能会访问位于同步起始位置以前的记录，为了避免报错，可以在下游插入必要的全量数据，或者在任务配置文件中启用 [Safe mode](glossary.md#safe-mode)。
> - 如果目标表的建表 DDL 语句在同步起始位置之后，可以正常同步。

## 模拟上游插入数据

在开始迁移之前，我们向上游 MySQL 插入一些测试数据。假设 `log.messages` 的表结构如下：

{{< copyable "sql" >}}

```sql
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

向表中插入以下数据：

{{< copyable "sql" >}}

```sql
INSERT INTO messages VALUES (1, 'msg1'), (2, 'msg2'), (3, 'msg3');
```

## 迁移操作

### 确定增量同步起始位置

首先需要确定开始迁移的上游 binlog 位置。如果你确定 binlog 的同步位置，那么可以跳过这一步。

如果从当前位置开始同步 binlog，则可以使用 `SHOW MASTER STATUS` 命令查看当前位置：

```sql
MySQL [log]> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000001 |     2022 |              |                  | 09bec856-ba95-11ea-850a-58f2b4af5188:1-9 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.000 sec)
```

本例将从 `(mysql-bin.000001, 2022)` 这个位置开始同步。

如果需要从其他位置开始同步，可以使用 `SHOW BINLOG EVENTS` 语句，或者使用 `mysqlbinlog` 工具查看 binlog，选择合适的位置。

### 在下游创建表

由于建表 SQL 语句在同步起始位置之前，本次增量同步任务并不会自动在下游创建表。因此需要手动在上游使用 `SHOW CREATE TABLE` 查看表结构，并在下游创建该表。示例如下：

{{< copyable "sql" >}}

```sql
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

### 启动 DM 集群

可以参照 [快速上手](quick-start-with-dm.md#使用-binary-包部署-dm) 使用 binary 包启动 DM 集群快速验证，或者参照[使用 TiUP 部署 DM 集群](deploy-a-dm-cluster-using-tiup.md)进行部署。

### 创建上游数据源

1. 创建上游数据源配置文件 `source.yaml`，写入上游数据库的连接参数和其他配置。示例如下：

   {{< copyable "yaml" >}}

   ```yaml
   source-id: "mysql-01" # 数据源唯一 ID，在其他命令中标识数据源

   from:
     host: 127.0.0.1
     port: 3306
     user: "root"
     password: "" # 推荐使用 dmctl 对上游数据库的用户密码加密之后的密码
   ```

2. 使用 `operate-source create` 命令创建数据源：

   {{< copyable "shell-regular" >}}

   ```bash
   bin/dmctl --master-addr=127.0.0.1:8261 operate-source create source.yaml
   ```

   ```
   {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-01",
               "worker": "127.0.0.1:8262"
           }
       ]
   }
   ```

3. （可选）使用 `operate-source show` 命令查看当前数据源列表：

   {{< copyable "shell-regular" >}}

   ```bash
   bin/dmctl --master-addr=127.0.0.1:8261 operate-source show
   ```

   ```
   {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-01",
               "worker": "127.0.0.1:8262"
           }
       ]
   }
   ```

### 创建同步任务

1. 创建任务配置文件 `task.yaml`，配置增量同步模式，以及每个上游的同步起点。示例如下：

   {{< copyable "yaml" >}}
 
   ```yaml
   name: test             # 任务名称，需要全局唯一
   task-mode: incremental # 任务模式，可设为 "full"、"incremental"、"all"
    
   mysql-instances:
     - source-id: "mysql-01" # 上游实例 ID
       meta:                 # `task-mode` 为 `incremental` 且下游数据库的 `checkpoint` 不存在时 binlog 迁移开始的位置; 如果 `checkpoint` 存在，则以 `checkpoint` 为准
         binlog-name: mysql-bin.000001
         binlog-pos: 2022
   ```

2. 配置需要同步的库：

   {{< copyable "yaml" >}}

   ```yaml
   block-allow-list:   # 上游数据库实例匹配的表的 block-allow-list 过滤规则集，如果 DM 版本早于 v2.0.0-beta.2 则使用 black-white-list
     bw-rule-1:        # 黑白名单配置项 ID
       do-dbs: ["log"] # 迁移哪些库
   
   mysql-instances:
     - source-id: "mysql-01"          # 上游实例或者复制组 ID
       block-allow-list:  "bw-rule-1" # 黑白名单配置名称，如果 DM 版本早于 v2.0.0-beta.2 则使用 black-white-list
   ```

3. 补充下游数据库连接信息：

   {{< copyable "yaml" >}}

   ```yaml
   target-database:       # 下游数据库实例配置
     host: "127.0.0.1"
     port: 4000
     user: "root"
     password: ""         # 如果密码不为空，则推荐使用经过 dmctl 加密的密文
   ```

4. 启用 [Safe mode](glossary.md#safe-mode) 避免下游访问未同步记录报错

   {{< copyable "yaml" >}}

   ```yaml
   syncers:            # sync 处理单元的运行配置参数
     global:           # 配置名称
       safe-mode: true # 设置为 true，则将来自上游的 `INSERT` 改写为 `REPLACE`，将 `UPDATE` 改写为 `DELETE` 与 `REPLACE`，保证在表结构中存在主键或唯一索引的条件下迁移数据时可以重复导入 DML。在启动或恢复增量复制任务的前 5 分钟内 TiDB DM 会自动启动 safe mode
   # ----------- 实例配置 -----------
   mysql-instances:
     - source-id: "mysql-01"         # 上游实例或者复制组 ID
       syncer-config-name: "global"  # syncers 配置的名称
   ```

   完整的任务配置文件如下：

   {{< copyable "yaml" >}}

   ```yaml
   name: test             # 任务名称，需要全局唯一
   task-mode: incremental # 任务模式，可设为 "full"、"incremental"、"all"
   
   target-database:       # 下游数据库实例配置
     host: "127.0.0.1"
     port: 4000
     user: "root"
     password: ""         # 如果密码不为空，则推荐使用经过 dmctl 加密的密文
   
   ## ******** 功能配置集 **********
   block-allow-list:   # 上游数据库实例匹配的表的 block-allow-list 过滤规则集，如果 DM 版本早于 v2.0.0-beta.2 则使用 black-white-list
     bw-rule-1:        # 黑白名单配置项 ID
       do-dbs: ["log"] # 迁移哪些库
   syncers:            # sync 处理单元的运行配置参数
     global:           # 配置名称
       safe-mode: true # 设置为 true，则将来自上游的 `INSERT` 改写为 `REPLACE`，将 `UPDATE` 改写为 `DELETE` 与 `REPLACE`，保证在表结构中存在主键或唯一索引的条件下迁移数据时可以重复导入 DML。在启动或恢复增量复制任务的前 5 分钟内 TiDB DM 会自动启动 safe mode
   
   # ----------- 实例配置 -----------
   mysql-instances:
     - source-id: "mysql-01"         # 上游实例或者复制组 ID
       block-allow-list: "bw-rule-1" # 黑白名单配置名称，如果 DM 版本早于 v2.0.0-beta.2 则使用 black-white-list
       syncer-config-name: "global"  # syncers 配置的名称
       meta:                         # `task-mode` 为 `incremental` 且下游数据库的 `checkpoint` 不存在时 binlog 迁移开始的位置; 如果 `checkpoint` 存在，则以 `checkpoint` 为 准
         binlog-name: mysql-bin.000001
         binlog-pos: 2022
   ```

5. 使用 `start-task` 命令创建同步任务：

   {{< copyable "shell-regular" >}}

   ```bash
   bin/dmctl --master-addr=127.0.0.1:8261 start-task task.yaml
   ```

   ```
   {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-01",
               "worker": "127.0.0.1:8262"
           }
       ]
   }
   ```

6. 使用 `query-status` 查看同步任务，确认无报错信息：

   {{< copyable "shell-regular" >}}

   ```bash
   bin/dmctl --master-addr=127.0.0.1:8261 query-status test
   ```

   ```
   {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "sourceStatus": {
                   "source": "mysql-01",
                   "worker": "127.0.0.1:8262",
                   "result": null,
                   "relayStatus": null
               },
               "subTaskStatus": [
                   {
                       "name": "test",
                       "stage": "Running",
                       "unit": "Sync",
                       "result": null,
                       "unresolvedDDLLockID": "",
                       "sync": {
                           "totalEvents": "0",
                           "totalTps": "0",
                           "recentTps": "0",
                           "masterBinlog": "(mysql-bin.000001, 2022)",
                           "masterBinlogGtid": "09bec856-ba95-11ea-850a-58f2b4af5188:1-9",
                           "syncerBinlog": "(mysql-bin.000001, 2022)",
                           "syncerBinlogGtid": "",
                           "blockingDDLs": [
                           ],
                           "unresolvedGroups": [
                           ],
                           "synced": true,
                           "binlogType": "remote"
                       }
                   }
               ]
           }
       ]
   }
   ```

## 测试同步任务

在上游数据库插入新增数据：

{{< copyable "sql" >}}

```sql
MySQL [log]> INSERT INTO messages VALUES (4, 'msg4'), (5, 'msg5');
Query OK, 2 rows affected (0.010 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

此时上游数据为：

```sql
MySQL [log]> SELECT * FROM messages;
+----+---------+
| id | message |
+----+---------+
|  1 | msg1    |
|  2 | msg2    |
|  3 | msg3    |
|  4 | msg4    |
|  5 | msg5    |
+----+---------+
5 rows in set (0.001 sec)
```

查询下游数据库，可以发现 `(3, 'msg3')` 之后的数据已同步成功：

{{< copyable "" >}}

```sql
MySQL [log]> SELECT * FROM messages;
+----+---------+
| id | message |
+----+---------+
|  4 | msg4    |
|  5 | msg5    |
+----+---------+
2 rows in set (0.001 sec)
```
