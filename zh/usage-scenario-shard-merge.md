---
title: DM 分库分表合并场景
aliases: ['/docs-cn/tidb-data-migration/dev/usage-scenario-shard-merge/']
---

# DM 分库分表合并场景

本文介绍如何在分库分表合并场景中使用 Data Migration (DM)。使用场景中，三个上游 MySQL 实例的分库和分表数据需要迁移至下游 TiDB 集群。

## 上游实例

假设上游库结构如下：

- 实例 1

    | Schema | Tables |
    |:------|:------|
    | user  | information, log_north, log_bak |
    | store_01 | sale_01, sale_02 |
    | store_02 | sale_01, sale_02 |

- 实例 2

    | Schema | Tables |
    |:------|:------|
    | user  | information, log_east, log_bak |
    | store_01 | sale_01, sale_02 |
    | store_02 | sale_01, sale_02 |

- 实例 3

    | Schema | Tables |
    |:------|:------|
    | user  | information, log_south, log_bak |
    | store_01 | sale_01, sale_02 |
    | store_02 | sale_01, sale_02 |

## 迁移需求

1. 同名表合并场景，比如将三个实例中的 `user`.`information` 表合并至下游 TiDB 中的 `user`.`information` 表。
2. 不同名表合并场景，比如将三个实例中的 `user`.`log_{north|south|east}` 表合并至下游 TiDB 中的 `user`.`log_{north|south|east}` 表。
3. 分片表合并场景，比如将三个实例中的 `store_{01|02}`.`sale_{01|02}` 表合并至下游 TiDB 中的 `store`.`sale` 表。
4. 过滤删除操作场景，比如过滤掉三个实例中 `user`.`log_{north|south|east}` 表的所有删除操作。
5. 过滤删除操作场景，比如过滤掉三个实例中 `user`.`information` 表的所有删除操作。
6. 过滤删除操作场景，比如过滤掉三个实例中 `store_{01|02}`.`sale_{01|02}` 表的所有删除操作。
7. 使用通配符过滤特定表的场景，比如使用通配符 `user`.`log_*` 过滤掉三个实例的 `user`.`log_bak` 表。
8. 主键冲突处理场景，假设 `store_{01|02}`.`sale_{01|02}` 表带有 bigint 型的自增主键，将其合并至 TiDB 时会引发冲突，可以使用相应的方案来避免冲突。

## 下游实例

假设迁移后下游库结构如下：

| Schema | Tables |
|:------|:------|
| user | information, log_north, log_east, log_south|
| store | sale |

## 迁移方案

- 要满足迁移需求 #1 和 #2，配置 [Table routing 规则](key-features.md#table-routing) 如下：

    {{< copyable "" >}}

    ```yaml
    routes:
      ...
      user-route-rule:
        schema-pattern: "user"
        target-schema: "user"
    ```

- 要满足迁移需求 #3，配置 [table routing 规则](key-features.md#table-routing) 如下：

    {{< copyable "" >}}

    ```yaml
    routes:
      ...
      store-route-rule:
        schema-pattern: "store_*"
        target-schema: "store"
      sale-route-rule:
        schema-pattern: "store_*"
        table-pattern: "sale_*"
        target-schema: "store"
        target-table:  "sale"
    ```

- 要满足迁移需求 #4 和 #5，配置 [Binlog event filter 规则](key-features.md#binlog-event-filter) 如下：

    {{< copyable "" >}}

    ```yaml
    filters:
      ...
      user-filter-rule:
        schema-pattern: "user"
        events: ["truncate table", "drop table", "delete", "drop database"]
        action: Ignore
    ```

    > **注意：**
    >
    > 迁移需求 #4、#5 的操作意味着过滤掉所有对 `user` 库的删除操作，所以此处配置了库级别的过滤规则，`user` 库以后参与复制的表的所有删除操作也都会被过滤。

- 要满足迁移需求 #6，配置 [Binlog event filter 规则](key-features.md#binlog-event-filter) 如下：

    {{< copyable "" >}}

    ```yaml
    filters:
      ...
      sale-filter-rule:
        schema-pattern: "store_*"
        table-pattern: "sale_*"
        events: ["truncate table", "drop table", "delete"]
        action: Ignore
      store-filter-rule:
        schema-pattern: "store_*"
        events: ["drop database"]
        action: Ignore
    ```

- 要满足迁移需求 #7，配置 [Block & Allow Lists](key-features.md#block--allow-table-lists) 如下：

    {{< copyable "" >}}

    ```yaml
    block-allow-list:    # 如果 DM 版本 <= v2.0.0-beta.2 则使用 black-white-list
      log-bak-ignored:
        ignore-tables:
        - db-name: "user"
          tbl-name: "log_bak"
    ```

- 要满足迁移需求 #8，首先参考[自增主键冲突处理](shard-merge-best-practices.md#自增主键冲突处理)来解决冲突，保证在迁移到下游时不会因为分表中有相同的主键值而使迁移出现异常，然后需要配置 `ignore-checking-items` 来跳过自增主键冲突的检查：

    {{< copyable "" >}}

    ```yaml
    ignore-checking-items: ["auto_increment_ID"]
    ```

## 迁移任务配置

迁移任务的完整配置如下。详情请参阅 [Data Migration 任务配置文件](task-configuration-file.md)。

{{< copyable "" >}}

```yaml
name: "shard_merge"
task-mode: all
meta-schema: "dm_meta"
remove-meta: false
ignore-checking-items: ["auto_increment_ID"]

target-database:
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: ""

mysql-instances:
  -
    source-id: "instance-1"
    route-rules: ["user-route-rule", "store-route-rule", "sale-route-rule"]
    filter-rules: ["user-filter-rule", "store-filter-rule", "sale-filter-rule"]
    block-allow-list:  "log-bak-ignored"     # 如果 DM 版本 <= v2.0.0-beta.2 则使用 black-white-list
    mydumper-config-name: "global"
    loader-config-name: "global"
    syncer-config-name: "global"

  -
    source-id: "instance-2"
    route-rules: ["user-route-rule", "store-route-rule", "sale-route-rule"]
    filter-rules: ["user-filter-rule", "store-filter-rule", "sale-filter-rule"]
    block-allow-list:  "log-bak-ignored"    # 如果 DM 版本 <= v2.0.0-beta.2 则使用 black-white-list
    mydumper-config-name: "global"
    loader-config-name: "global"
    syncer-config-name: "global"
  -
    source-id: "instance-3"
    route-rules: ["user-route-rule", "store-route-rule", "sale-route-rule"]
    filter-rules: ["user-filter-rule", "store-filter-rule", "sale-filter-rule"]
    block-allow-list:  "log-bak-ignored"    # 如果 DM 版本 <= v2.0.0-beta.2 则使用 black-white-list
    mydumper-config-name: "global"
    loader-config-name: "global"
    syncer-config-name: "global"

# 所有实例共享的其他通用配置

routes:
  user-route-rule:
    schema-pattern: "user"
    target-schema: "user"
  store-route-rule:
    schema-pattern: "store_*"
    target-schema: "store"
  sale-route-rule:
    schema-pattern: "store_*"
    table-pattern: "sale_*"
    target-schema: "store"
    target-table:  "sale"

filters:
  user-filter-rule:
    schema-pattern: "user"
    events: ["truncate table", "drop table", "delete", "drop database"]
    action: Ignore
  sale-filter-rule:
    schema-pattern: "store_*"
    table-pattern: "sale_*"
    events: ["truncate table", "drop table", "delete"]
    action: Ignore
  store-filter-rule:
    schema-pattern: "store_*"
    events: ["drop database"]
    action: Ignore

block-allow-list:      # 如果 DM 版本 <= v2.0.0-beta.2 则使用 black-white-list
  log-bak-ignored:
    ignore-tables:
    - db-name: "user"
      tbl-name: "log_bak"

mydumpers:
  global:
    threads: 4
    chunk-filesize: 64
    skip-tz-utc: true

loaders:
  global:
    pool-size: 16
    dir: "./dumped_data"

syncers:
  global:
    worker-count: 16
    batch: 100
    max-retry: 100
```
