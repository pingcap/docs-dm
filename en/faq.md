---
title: TiDB Data Migration FAQ
summary: Learn about frequently asked questions (FAQs) about TiDB Data Migration (DM).
aliases: ['/docs/tidb-data-migration/dev/faq/']
---

# TiDB Data Migration FAQ

This document collects the frequently asked questions (FAQs) about TiDB Data Migration (DM).

## Does DM support migrating data from Alibaba RDS or other cloud databases?

Currently, DM only supports decoding the standard version of MySQL or MariaDB binlog. It has not been tested for Alibaba Cloud RDS or other cloud databases. If you are confirmed that its binlog is in standard format, then it is supported.

## Does the regular expression of the block and allow list in the task configuration support `non-capturing (?!)`?

Currently, DM does not support it and only supports the regular expressions of the Golang standard library. See regular expressions supported by Golang via [re2-syntax](https://github.com/google/re2/wiki/Syntax).

## If a statement executed upstream contains multiple DDL operations, does DM support such migration?

DM will attempt to split a single statement containing multiple DDL change operations into multiple statements containing only one DDL operation, but might not cover all cases. It is recommended to include only one DDL operation in a statement executed upstream, or verify it in the test environment. If it is not supported, you can file an [issue](https://github.com/pingcap/dm/issues) to the DM repository.

## How to handle incompatible DDL statements?

When you encounter a DDL statement unsupported by TiDB, you need to manually handle it using dmctl (skipping the DDL statement or replacing the DDL statement with a specified DDL statement). For details, see [Handle failed SQL statements](handle-failed-sql-statements.md).

> **Note:**
>
> Currently, TiDB is not compatible with all the DDL statements that MySQL supports. See [MySQL Compatibility](https://pingcap.com/docs/dev/reference/mysql-compatibility/#ddl).

## How to reset the data migration task?

When an exception occurs during data migration and the data migration task cannot be resumed, you need to reset the task and re-migrate the data:

1. Execute the `stop-task` command to stop the abnormal data migration task.

2. Purge the data migrated to the downstream.

3. Use one of the following ways to restart the data migration task.

   - Specify a new task name in the task configuration file. Then execute `start-task {task-config-file}`.
   - Execute `start-task --remove-meta {task-config-file}`.

## How to handle the error returned by the DDL operation related to the gh-ost table, after `online-ddl-scheme: "gh-ost"` is set?

```
[unit=Sync] ["error information"="{\"msg\":\"[code=36046:class=sync-unit:scope=internal:level=high] online ddls on ghost table `xxx`.`_xxxx_gho`\\ngithub.com/pingcap/dm/pkg/terror.(*Error).Generate ......
```

The above error can be caused by the following reason:

In the last `rename ghost_table to origin table` step, DM reads the DDL information in memory, and restores it to the DDL of the origin table.

However, the DDL information in memory is obtained in either of the two ways:

- DM [processes the gh-ost table during the `alter ghost_table` operation](feature-online-ddl-scheme.md#online-schema-change-gh-ost) and records the DDL information of `ghost_table`;
- When DM-worker is restarted to start the task, DM reads the DDL from `dm_meta.{task_name}_onlineddl`.

Therefore, in the process of incremental replication, if the specified Pos has skipped the `alter ghost_table` DDL but the Pos is still in the online-ddl process of gh-ost, the ghost_table is not written into memory or `dm_meta.{task_name}_onlineddl` correctly. In such cases, the above error is returned.

You can avoid this error by the following steps:

1. Remove the `online-ddl-scheme` configuration of the task.

2. Configure `_{table_name}_gho`, `_{table_name}_ghc`, and `_{table_name}_del` in `block-allow-list.ignore-tables`.

3. Execute the upstream DDL in the downstream TiDB manually.

4. After the Pos is replicated to the position after the gh-ost process, re-enable the `online-ddl-scheme` and comment out `block-allow-list.ignore-tables`.

## How to add tables to the existing data migration tasks?

If you need to add tables to a data migration task that is running, you can address it in the following ways according to the stage of the task.

> **Note:**
>
> Because adding tables to an existing data migration task is complex, it is recommended that you perform this operation only when necessary.

### In the `Dump` stage

Since MySQL cannot specify a snapshot for export, it does not support updating data migration tasks during the export and then restarting to resume the export through the checkpoint. Therefore, you cannot dynamically add tables that need to be migrated at the `Dump` stage.

If you really need to add tables for migration, it is recommended to restart the task directly using the new configuration file.

### In the `Load` stage

During the export, multiple data migration tasks usually have different binlog positions. If you merge the tasks in the `Load` stage, they might not be able to reach consensus on binlog positions. Therefore, it is not recommended to add tables to a data migration task in the `Load` stage.

### In the `Sync` stage

When the data migration task is in the `Sync` stage, if you add additional tables to the configuration file and restart the task, DM does not re-execute full export and import for the newly added tables. Instead, DM continues incremental replication from the previous checkpoint.

Therefore, if the full data of the newly added table has not been imported to the downstream, you need to use a separate data migration task to export and import the full data to the downstream.

Record the position information in the global checkpoint (`is_global=1`) corresponding to the existing migration task as `checkpoint-T`, such as `(mysql-bin.000100, 1234)`. Record the position information of the full export `metedata` (or the checkpoint of another data migration task in the `Sync` stage) of the table to be added to the migration task as `checkpoint-S`, such as `(mysql-bin.000099, 5678)`. You can add the table to the migration task by the following steps:

1. Use `stop-task` to stop an existing migration task. If the table to be added belongs to another running migration task, stop that task as well.

2. Use a MySQL client to connect the downstream TiDB database and manually update the information in the checkpoint table corresponding to the existing migration task to the smaller value between `checkpoint-T` and `checkpoint-S`. In this example, it is `(mysql- bin.000099, 5678)`.

    - The checkpoint table to be updated is `{task-name}_syncer_checkpoint` in the `{dm_meta}` schema.

    - The checkpoint rows to be updated match `id=(source-id)` and `is_global=1`.

    - The checkpoint columns to be updated are  `binlog_name` and `binlog_pos`.

3. Set `safe-mode: true` for the `syncers` in the task to ensure reentrant execution.

4. Start the task using `start-task`.

5. Observe the task status through `query-status`. When `syncerBinlog` exceeds the larger value of `checkpoint-T` and `checkpoint-S`, restore `safe-mode` to the original value and restart the task. In this example, it is `(mysql-bin.000100, 1234)`.

## How to handle the error `packet for query is too large. Try adjusting the 'max_allowed_packet' variable` that occurs during the full import?

Set the parameters below to a value larger than the default 67108864 (64M).

- The global variable of the TiDB server: `max_allowed_packet`.
- The configuration item in the task configuration file: `target-database.max-allowed-packet`. For details, refer to [DM Advanced Task Configuration File](task-configuration-file-full.md).

For details, see [Loader solution](https://docs.pingcap.com/tidb/stable/loader-overview#solution).

## How to handle the error `Error 1054: Unknown column 'binlog_gtid' in 'field list'` that occurs when existing DM migration tasks of an DM 1.0 cluster are running on a DM 2.0 cluster?

DM 2.0 introduces more fields to metadata tables such as checkpoint. In DM 2.0, if you directly run the `start-task` command with the task configuration file of the DM 1.0 cluster to continue the incremental data replication, the error `Error 1054: Unknown column 'binlog_gtid' in 'field list'` occurs.

This error can be handled in any of the following ways:

- [Import a DM 1.0 cluster into a new DM 2.0 cluster using TiUP](maintain-dm-using-tiup.md#import-and-upgrade-a-dm-10-cluster-deployed-using-dm-ansible).
- [Manually import DM migration tasks of a DM 1.0 cluster to a DM 2.0 cluster](manually-upgrade-dm-1.0-to-2.0.md)。
