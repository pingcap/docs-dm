---
title: DM Advanced Task Configuration File
aliases: ['/tidb-data-migration/stable/task-configuration-file-full','/docs/tidb-data-migration/stable/task-configuration-file-full/','/docs/tidb-data-migration/v1.0/task-configuration-file-full/','/docs/dev/reference/tools/data-migration/configure/task-configuration-file-full/','/docs/v3.1/reference/tools/data-migration/configure/task-configuration-file-full/','/docs/v3.0/reference/tools/data-migration/configure/task-configuration-file-full/','/docs/v2.1/reference/tools/data-migration/configure/task-configuration-file-full/','/docs/stable/reference/tools/data-migration/configure/task-configuration-file-full/']
---

# DM Advanced Task Configuration File

This document introduces the advanced task configuration file of Data Migration --
[`task_advanced.yaml`](https://github.com/pingcap/dm/blob/master/dm/master/task_advanced.yaml), including [global configuration](#global-configuration) and [instance configuration](#instance-configuration).

For the feature and configuration of each configuration item, see [Data migration features](feature-overview.md).

## Important concepts

For description of important concepts including `source-id` and the DM-worker ID, see [Important concepts](config-overview.md#important-concepts).

## Disable checking items

DM checks items according to the task type, see [Disable checking items](precheck.md#disable-checking-items). You can use `ignore-checking-items` in the task configuration file to disable checking items.

## Task configuration file template (advanced)

The following is the task configuration file template which allows you to perform **advanced** data migration tasks.

```yaml
---

# ----------- Global setting -----------
## ********* Basic configuration *********
name: test                      # The name of the task. Should be globally unique.
task-mode: all                  # The task mode. Can be set to `full`/`incremental`/`all`.
is-sharding: true               # Whether it is a task to merge shards.
meta-schema: "dm_meta"          # The downstream database that stores the `meta` information.
remove-meta: false              # Whether to remove the `meta` information (`checkpoint` and `onlineddl`) corresponding to the task name before starting the migration task.
enable-heartbeat: false         # Whether to enable the heartbeat feature.
online-ddl-scheme: "gh-ost"     # Only "gh-ost" and "pt" are currently supported.
case-sensitive: false           # Whether schema/table is case-sensitive.
ignore-checking-items: []       # No element, which means not to disable any checking items.
clean-dump-file: true           # Whether to clean up the files generated during data dump. Note that these include `metadata` files. New in v1.0.7.

target-database:                # Configuration of the downstream database instance.
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: "/Q7B9DizNLLTTfiZHv9WoEAKamfpIUs="  # It is recommended to use a password encrypted with dmctl
  session:                                      # The session variables of TiDB, supported since v1.0.6. For details, go to `https://pingcap.com/docs/stable/system-variables`
    sql_mode: "ANSI_QUOTES,NO_ZERO_IN_DATE,NO_ZERO_DATE"
    tidb_skip_utf8_check: 1
    tidb_constraint_check_in_place: 0


## ******** Feature configuration set **********
# The routing mapping rule set between the upstream and downstream tables.
routes:
  route-rule-1:                   # The name of the routing mapping rule
    schema-pattern: "test_*"      # The pattern of the upstream schema name, wildcard characters (*?) are supported
    table-pattern: "t_*"          # The pattern of the upstream table name, wildcard characters (*?) are supported
    target-schema: "test"         # The name of the downstream schema
    target-table: "t"             # The name of the downstream table
  route-rule-2:
    schema-pattern: "test_*"
    target-schema: "test"

# The binlog event filter rule set of the matched table of the upstream database instance.
filters:
  filter-rule-1:                                # The name of the filtering rule
    schema-pattern: "test_*"                    # The pattern of the upstream schema name, wildcard characters (*?) are supported
    table-pattern: "t_*"                        # The pattern of the upstream schema name, wildcard characters (*?) are supported
    events: ["truncate table", "drop table"]    # What event types to match
    action: Ignore                              # Whether to replicate (Do) or ignore (Ignore) the binlog that matches the filtering rule
  filter-rule-2:
    schema-pattern: "test_*"
    events: ["all dml"]
    action: Do

# The filter rule set of the block and allow list of the matched table of the upstream database instance.
block-allow-list:                    # Use black-white-list if the DM's version <= v1.0.6.
  bw-rule-1:                         # The name of the block and allow list rule
    do-dbs: ["~^test.*", "user"]     # The allow list of upstream schemas needs to be migrated
    ignore-dbs: ["mysql", "account"] # The block list of upstream schemas needs to be migrated
    do-tables:                       # The allow list of upstream tables needs to be migrated
    - db-name: "~^test.*"
      tbl-name: "~^t.*"
    - db-name: "user"
      tbl-name: "information"
    ignore-tables:                   # The block list of upstream tables needs to be migrated
    - db-name: "user"
      tbl-name: "log"

# Configuration arguments of the dump processing unit
mydumpers:
  global:                            # The configuration name of the processing unit.
    # The binary file path of the dump unit ("./bin/mydumper" by default).
    mydumper-path: "./bin/mydumper"
    threads: 4                       # The number of the threads that the dump unit dumps from the upstream database (4 by default).
    chunk-filesize: 64               # The size of the file generated by the dump unit (64 in MB by default).
    skip-tz-utc: true                # Ignore timezone conversion for time type data (true by default).
    extra-args: "--no-locks"         # Other arguments of the dump unit. In v1.0.2 and later versions, DM automatically generates the table-list configurable items.

# Configuration arguments of the load processing unit
loaders:
  global:                            # The configuration name of the processing unit.
    pool-size: 16                    # The number of threads that concurrently execute dumped SQL files in the load unit (16 by default).
    # The directory that the load unit reads from and the dump unit outputs SQL files to  ("./dumped_data" by default). Directories for different tasks of the same instance must be different.
    dir: "./dumped_data"

# Configuration arguments of the sync processing unit
syncers:
  global:                            # The configuration name of the processing unit.
    worker-count: 16                 # The number of threads that replicate binlog events concurrently in the sync unit.
    batch: 100                       # The number of SQL statements in a transaction batch that the sync unit replicates to the downstream database (100 by default).
    enable-ansi-quotes: true         # Enable this argument if `sql-mode: "ANSI_QUOTES"` is set in the `session`
    safe-mode: false                 # If set to true, `INSERT` statements from upstream are rewritten to `REPLACE` statements, and `UPDATE` statements are rewritten to `DELETE` and `REPLACE` statements. This ensures that DML statements can be imported repeatedly during data migration when there is any primary key or unique index in the table schema. TiDB DM automatically enables safe mode within the first 5 minutes after starting or resuming migration tasks.

# ----------- Instance configuration -----------
mysql-instances:
  -
    source-id: "mysql-replica-01"                                      # The ID of the upstream instance or replication group. It can be configured by referring to the `source_id` in the `inventory.ini` file or the `source-id` in the `dm-master.toml` file.
    meta:                                                              # The position where the binlog replication starts when `task-mode` is `incremental` and the downstream database checkpoint does not exist. If the checkpoint exists, the checkpoint is used.

      binlog-name: binlog.000001
      binlog-pos: 4

    route-rules: ["route-rule-1", "route-rule-2"]                      # The name of the mapping rule between the table matching the upstream database instance and the downstream database.
    filter-rules: ["filter-rule-1"]                                    # The name of the binlog event filtering rule of the table matching the upstream database instance.
    block-allow-list:  "bw-rule-1"                                     # The name of the block and allow lists filtering rule of the table matching the upstream database instance. Use black-white-list if the DM's version <= v1.0.6.

    mydumper-config-name: "global"                                     # The configuration name of the dump processing unit.
    loader-config-name: "global"                                       # The configuration name of the load processing unit.
    syncer-config-name: "global"                                       # The configuration name of the sync processing unit.

  -
    source-id: "mysql-replica-02"
    mydumper-config-name: "global"
    loader-config-name: "global"
    syncer-config-name: "global"
```

## Configuration order

1. Edit the [global configuration](#global-configuration).
2. Edit the [instance configuration](#instance-configuration) based on the global configuration.

## Global configuration

### Basic configuration

Refer to the comments in the [template](#task-configuration-file-template-advanced) to see more details. Detailed explanations about `task-mode` are as follows:

- Description: the task mode that can be used to specify the data migration task to be executed.
- Value: string (`full`, `incremental`, or `all`).
    - `full` only makes a full backup of the upstream database and then imports the full data to the downstream database.
    - `incremental`: Only replicates the incremental data of the upstream database to the downstream database using the binlog. You can set the `meta` configuration item of the instance configuration to specify the starting position of incremental replication.
    - `all`: `full` + `incremental`. Makes a full backup of the upstream database, imports the full data to the downstream database, and then uses the binlog to make an incremental replication to the downstream database starting from the exported position during the full backup process (binlog position).

### Feature configuration set

Arguments in each feature configuration set are explained in the comments in the [template](#task-configuration-file-template-advanced).

| Parameter        | Description                                    |
| :------------ | :--------------------------------------- |
| `routes` | The routing mapping rule set between the upstream and downstream tables. If the names of the upstream and downstream schemas and tables are the same, this item does not need to be configured. See [Table Routing](feature-overview.md#table-routing) for usage scenarios and sample configurations. |
| `filters` | The binlog event filter rule set of the matched table of the upstream database instance. If binlog filtering is not required, this item does not need to be configured. See [Binlog Event Filter](feature-overview.md#binlog-event-filter) for usage scenarios and sample configurations. |
| `block-allow-list` | The filter rule set of the block and allow list of the matched table of the upstream database instance. It is recommended to specify the schemas and tables that need to be migrated through this item, otherwise all schemas and tables are migrated. See [Binlog Event Filter](feature-overview.md#binlog-event-filter)[Block & Allow Lists](feature-overview.md#block-and-allow-table-lists) for usage scenarios and sample configurations. |
| `mydumpers` | Configuration arguments of the dump processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `thread` only using `mydumper-thread`. |
| `loaders` | Configuration arguments of the load processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `pool-size` only using `loader-thread`. |
| `syncers` | Configuration arguments of the sync processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `worker-count` only using `syncer-thread`. |

## Instance configuration

This part defines the subtask of data migration. DM supports migrating data from one or multiple MySQL instances in the upstream to the same instance in the downstream.

For the configuration details of the above options, see the corresponding part in [Feature configuration set](#feature-configuration-set), as shown in the following table.

| Option | Corresponding part |
| :------ | :------------------ |
| `route-rules` | `routes` |
| `filter-rules` | `filters` |
| `block-allow-list` | `block-allow-list` |
| `mydumper-config-name` | `mydumpers` |
| `loader-config-name` | `loaders` |
| `syncer-config-name` | `syncers`  |
