---
title: Data Migration Task Configuration File
summary: This document introduces the task configuration file of Data Migration.
aliases: ['/docs/tidb-data-migration/stable/task-configuration-file/','/docs/tidb-data-migration/v1.0/task-configuration-file/','/docs/dev/reference/tools/data-migration/configure/task-configuration-file','/docs/v3.1/reference/tools/data-migration/configure/task-configuration-file','/docs/v3.0/reference/tools/data-migration/configure/task-configuration-file','/docs/v2.1/reference/tools/data-migration/configure/task-configuration-file','/docs/tools/dm/dm-configuration-file-overview/']
---

# Data Migration Task Configuration File

This document introduces the basic task configuration file of Data Migration --
[`task_basic.yaml`](https://github.com/pingcap/dm/blob/master/dm/master/task_advanced.yaml), including [global configuration](#global-configuration) and [instance configuration](#instance-configuration).

DM also implements [an advanced task configuration file](task-configuration-file-full.md) which provides greater flexibility and more control over DM.

For the feature and configuration of each configuration item, see [Data migration features](feature-overview.md).

## Important concepts

For description of important concepts including `source-id` and the DM-worker ID, see [Important concepts](config-overview.md#important-concepts).

## Task configuration file template (basic)

The following is a task configuration file template which allows you to perform basic data migration tasks.

```yaml
---

# ----------- Global configuration -----------
## ********** Basic configuration ************

```yaml
name: test                      # The name of the task. Should be globally unique.
task-mode: all                  # The task mode. Can be set to `full`/`incremental`/`all`.

target-database:                # Configuration of the downstream database instance.
  host: "127.0.0.1"
  port: 4000
  user: "root"
  password: ""                  # The dmctl encryption is needed when the password is not empty.

## ******** Feature configuration set **********
# The filter rule set of the block and allow list of the matched table of the upstream database instance.
block-allow-list:        # Use black-white-list if the DM's version <= v1.0.6.
  bw-rule-1:             # The name of the block and allow lists filtering rule of the table matching the upstream database instance.
    do-dbs: ["all_mode"] # Allow list of upstream tables needs to be migrated
# ----------- Instance configuration -----------
mysql-instances:
  # The ID of the upstream instance or migration group. It can be configured by referring to the `source-id` in the `dm-master.toml` file.
  - source-id: "mysql-replica-01"
    block-allow-list:  "bw-rule-1"     # Use black-white-list if the DM's version <= v1.0.6.
        mydumper-thread: 4             # The number of threads that the dump unit uses for dumping data, new in v1.0.2 and later versions
        loader-thread: 16              # The number of threads that the load unit uses for loading data, new in v1.0.2 and later versions
        syncer-thread: 16              # The number of threads that the sync unit uses for replicating incremental data, new in v1.0.2 and later versions
  - source-id: "mysql-replica-02"
    block-allow-list:  "bw-rule-1"     # Use black-white-list if the DM's version <= v1.0.6.
    mydumper-thread: 4
    loader-thread: 16
    syncer-thread: 16
```

## Configuration order

1. Edit the [global configuration](#global-configuration).
2. Edit the [instance configuration](#instance-configuration) based on the global configuration.

## Global configuration

### Basic configuration

Refer to the comments in the [template](#task-configuration-file-template-basic) to see more details. Specific instruction about `task-mode` are as follows:

- Description: the task mode that can be used to specify the data migration task to be executed.
- Value: string (`full`, `incremental`, or `all`).
    - `full` only makes a full backup of the upstream database and then imports the full data to the downstream database.
    - `incremental`: Only replicates the incremental data of the upstream database to the downstream database using the binlog. You can set the `meta` configuration item of the instance configuration to specify the starting position of incremental replication.
    - `all`: `full` + `incremental`. Makes a full backup of the upstream database, imports the full data to the downstream database, and then uses the binlog to make an incremental replication to the downstream database starting from the exported position during the full backup process (binlog position).

### Feature configuration set

For basic applications, you only need to modify the block and allow lists filtering rule. Refer to the comments about `block-allow-list` in the [template](#task-configuration-file-template-basic) or [Block & allow table lists](feature-overview.md#block-and-allow-table-lists) to see more details.

## Instance configuration

This part defines the subtask of data migration. DM supports migrating data from one or multiple MySQL instances to the same instance.

For more details, refer to the comments about `mysql-instances` in the [template](#task-configuration-file-template-basic).

## Modify the task configuration

In some cases, you might need to update the task configuration. For example, if you set `remove-meta` to `true` and `task-mode` to `all` when resetting the data migration task, you need to set `remove-meta` to `false` after the task is reset. This can prevent the task from being migrated the next time the task is started.

It is recommended to update the modified configuration to the DM cluster by executing the `stop-task` and `start-task` commands, since the DM cluster persists the task configuration. If the task configuration file is modified directly, without restarting the task, the configuration changes does not take effect. In this case, the DM cluster still reads the previous task configuration when the DM cluster is restarted.

To illustrate how to modify the task configuration, the following is an example of modifying `remove-meta`:

1. Modify the task configuration file and set `remove-meta` to `false`.

2. Stop the task by executing the `stop-task` command:

    {{< copyable "" >}}

    ```bash
    stop-task <task-name | task-file>
    ```

3. Start the task by executing the `start-task` command:

    {{< copyable "" >}}

    ```bash
    start-task <config-file>
    ```
