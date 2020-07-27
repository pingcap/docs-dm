---
title: DM Advanced Task Configuration File
aliases: ['/docs/tidb-data-migration/dev/task-configuration-file-full/']
---

# DM Advanced Task Configuration File

This document introduces the advanced task configuration file of Data Migration (DM), including [global configuration](#global-configuration) and [instance configuration](#instance-configuration).

For the feature and configuration of each configuration item, see [Data replication features](key-features.md).

## Important concepts

For description of important concepts including `source-id` and the DM-worker ID, see [Important concepts](config-overview.md#important-concepts).

## Task configuration file template (advanced)

The following is the task configuration file template which allows you to perform **advanced** data replication tasks.

```yaml
---

# ----------- Global setting -----------
## ********* Basic configuration *********
name: test                      # The name of the task. Should be globally unique.
task-mode: all                  # The task mode. Can be set to `full`/`incremental`/`all`.
is-sharding: true               # Whether it is a task to merge shards.
shard-mode: "optimistic"        # The shard DDL replication mode. Can be set to ""/`pessimistic`/`optimistic`.
ignore-checking-items: []       # The checking items that are ignored, including `all`/`dump_privilege`/`replication_privilege`/`version`/`binlog_enable`/`binlog_format`/`binlog_row_image`/`table_schema`/`schema_of_shard_tables`/`auto_increment_ID`.
meta-schema: "dm_meta"          # The downstream database that stores the `meta` information.
remove-meta: false              # Whether to remove the `meta` information (`checkpoint` and `onlineddl`) corresponding to the task name before starting the replication task.
enable-heartbeat: false         # If the heartbeat feature is enabled, DM regularly updates the MySQL heartbeat table in the upstream to estimate the replication delay.
heartbeat-update-interval: 1    # The interval at which DM updates the MySQL heartbeat table in the upstream.
heartbeat-report-interval: 10   # The interval at which DM estimates the lag (the delay time).
timezone: "Asia/Shanghai"       # The timezone.
case-sensitive: false           # Determines whether the schema/table is case-sensitive.
online-ddl-scheme: "gh-ost"     # Only "gh-ost" and "pt" are currently supported.
clean-dump-file: true           # Whether to clean up the files generated during data dump. Note that these include `metadata` files.

target-database:                # Configuration of the downstream database instance.
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: "/Q7B9DizNLLTTfiZHv9WoEAKamfpIUs="  # It is recommended to use a password encrypted with dmctl
  session:                                      # The session variables of TiDB, supported since 1.0.6. For details, see https://pingcap.com/docs/stable/system-variables/
    sql_mode: "ANSI_QUOTES,NO_ZERO_IN_DATE,NO_ZERO_DATE"
    tidb_skip_utf8_check: 1
    tidb_constraint_check_in_place: 0


## ******** Feature configuration set **********
# The routing mapping rule set between the upstream and downstream tables.
routes:
  route-rule-1:                 # The name of the routing mapping rule.
    schema-pattern: "test_*"    # The pattern of the upstream schema name, wildcard characters (*?) are supported.
    table-pattern: "t_*"        # The pattern of the upstream table name, wildcard characters (*?) are supported.
    target-schema: "test"       # The name of the downstream schema.
    target-table: "t"           # The name of the downstream table.
  route-rule-2:
    schema-pattern: "test_*"
    target-schema: "test"

# The binlog event filter rule set of the matched table of the upstream database instance.
filters:
  filter-rule-1:                                # The name of the filtering rule.
    schema-pattern: "test_*"                    # The pattern of the upstream schema name, wildcard characters (*?) are supported.
    table-pattern: "t_*"                        # The pattern of the upstream schema name, wildcard characters (*?) are supported.
    events: ["truncate table", "drop table"]    # What event types to match.
    action: Ignore                              # Whether to replicate (Do) or ignore (Ignore) the binlog that matches the filtering rule.
  filter-rule-2:
    schema-pattern: "test_*"
    events: ["all dml"]
    action: Do

# The filter rule set of the block allow list of the matched table of the upstream database instance.
block-allow-list:                    # Use black-white-list if the DM's version <= v2.0.0-beta.2.
  bw-rule-1:                         # The name of the block allow list rule.
    do-dbs: ["~^test.*", "user"]     # The allow list of upstream schemas needs to be replicated.
    ignore-dbs: ["mysql", "account"] # The block list of upstream schemas needs to be replicated.
    do-tables:                       # The allow list of upstream tables needs to be replicated.
    - db-name: "~^test.*"
      tbl-name: "~^t.*"
    - db-name: "user"
      tbl-name: "information"
    ignore-tables:                   # The block list of upstream tables needs to be replicated.
    - db-name: "user"
      tbl-name: "log"

# Configuration arguments of the Mydumper processing unit.
mydumpers:
  global:                            # The configuration name of the processing unit.
    threads: 4                       # The number of the threads Mydumper dumps from the upstream database (4 by default).
    chunk-filesize: 64               # The size of the file generated by Mydumper (64 in MB by default).
    skip-tz-utc: true                # Ignore timezone conversion for time type data (true by default).
    extra-args: "--no-locks"         # Other arguments of Mydumper. You do not need to manually configure table-list in `extra-args`, because table-list is automatically generated by DM.

# Configuration arguments of the Loader processing unit.
loaders:
  global:                            # The configuration name of the processing unit.
    pool-size: 16                    # The number of threads that execute Mydumper SQL files concurrently in Loader (16 by default).
    # The directory output by Mydumper that Loader reads ("./dumped_data" by default). Directories for different tasks of the same instance must be different (Mydumper outputs the SQL file based on the directory).
    dir: "./dumped_data"

# Configuration arguments of the Syncer processing unit.
syncers:
  global:                            # The configuration name of the processing unit.
    worker-count: 16                 # The number of threads that replicate binlog events concurrently in Syncer.
    batch: 100                       # The number of SQL statements in a transaction batch that Syncer replicates to the downstream database (100 by default).
    enable-ansi-quotes: true         # Enable this if `sql-mode: "ANSI_QUOTES"` is set in the `session`

# ----------- Instance configuration -----------
mysql-instances:
  -
    source-id: "mysql-replica-01"                   # The `source-id` in source.toml.
    meta:                                           # The position where the binlog replication starts when `task-mode` is `incremental` and the downstream database checkpoint does not exist. If the checkpoint exists, the checkpoint is used.

      binlog-name: binlog.000001
      binlog-pos: 4

    route-rules: ["route-rule-1", "route-rule-2"]   # The name of the mapping rule between the table matching the upstream database instance and the downstream database.
    filter-rules: ["filter-rule-1"]                 # The name of the binlog event filtering rule of the table matching the upstream database instance.
    block-allow-list:  "bw-rule-1"                  # The name of the block and allow lists filtering rule of the table matching the upstream database instance. Use black-white-list if the DM's version <= v2.0.0-beta.2.

    mydumper-config-name: "global"                  # The configuration name of the Mydumper processing unit.
    loader-config-name: "global"                    # The configuration name of the Loader processing unit.
    syncer-config-name: "global"                    # The configuration name of the Syncer processing unit.

  -
    source-id: "mysql-replica-02"                   # The `source-id` in source.toml.
    mydumper-thread: 4                              # The number of threads that Mydumper uses for dumping data. `mydumper-thread` corresponds to `threads` in the configuration of mydumper processing unit.
    loader-thread: 16                               # The number of threads that Loader uses for loading data. `loader-thread` corresponds to `pool-size` in the configuration of loader processing unit.
    syncer-thread: 16                               # The number of threads that Syncer uses for replicating incremental data. `syncer-thread` corresponds to `worker-count` in the configuration of the syncer processing unit.
```

## Configuration order

1. Edit the [global configuration](#global-configuration).
2. Edit the [instance configuration](#instance-configuration) based on the global configuration.

## Global configuration

### Basic configuration

Refer to the comments in the [template](#task-configuration-file-template-advanced) to see more details. Detailed explanations about `task-mode` are as follows:

- Description: the task mode that can be used to specify the data replication task to be executed.
- Value: string (`full`, `incremental`, or `all`).
    - `full` only makes a full backup of the upstream database and then imports the full data to the downstream database.
    - `incremental`: Only replicates the incremental data of the upstream database to the downstream database using the binlog. You can set the `meta` configuration item of the instance configuration to specify the starting position of incremental replication.
    - `all`: `full` + `incremental`. Makes a full backup of the upstream database, imports the full data to the downstream database, and then uses the binlog to make an incremental replication to the downstream database starting from the exported position during the full backup process (binlog position).

### Feature configuration set

Arguments in each feature configuration set are explained in the comments in the [template](#task-configuration-file-template-advanced).

| Parameter        | Description                                    |
| :------------ | :--------------------------------------- |
| `routes` | The routing mapping rule set between the upstream and downstream tables. If the names of the upstream and downstream schemas and tables are the same, this item does not need to be configured. See [Table Routing](key-features.md#table-routing) for usage scenarios and sample configurations. |
| `filters` | The binlog event filter rule set of the matched table of the upstream database instance. If binlog filtering is not required, this item does not need to be configured. See [Binlog Event Filter](key-features.md#binlog-event-filter) for usage scenarios and sample configurations. |
| `block-allow-list` | The filter rule set of the block allow list of the matched table of the upstream database instance. It is recommended to specify the schemas and tables that need to be replicated through this item, otherwise all schemas and tables are replicated. See [Binlog Event Filter](key-features.md#binlog-event-filter) and [Block & Allow Lists](key-features.md#block-and-allow-table-lists) for usage scenarios and sample configurations. |
| `mydumpers` | Configuration arguments of Mydumper processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `thread` only using `mydumper-thread`. |
| `loaders` | Configuration arguments of Loader processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `pool-size` only using `loader-thread`. |
| `syncers` | Configuration arguments of Syncer processing unit. If the default configuration is sufficient for your needs, this item does not need to be configured. Or you can configure `worker-count` only using `syncer-thread`. |

## Instance configuration

This part defines the subtask of data replication. DM supports replicating data from one or multiple MySQL instances in the upstream to the same instance in the downstream.

For the configuration details of the above options, see the corresponding part in [Feature configuration set](#feature-configuration-set), as shown in the following table.

| Option | Corresponding part |
| :------ | :------------------ |
| `route-rules` | `routes` |
| `filter-rules` | `filters` |
| `block-allow-list` | `block-allow-list` |
| `mydumper-config-name` | `mydumpers` |
| `loader-config-name` | `loaders` |
| `syncer-config-name` | `syncers`  |
