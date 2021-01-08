---
title: TiDB Data Migration FAQ
summary: Learn about frequently asked questions (FAQs) about TiDB Data Migration (DM).
aliases: ['/docs/tidb-data-migration/stable/faq/','/docs/tidb-data-migration/v1.0/faq/','/docs/dev/faq/data-migration/','/docs/dev/reference/tools/data-migration/faq/','/docs/v3.1/reference/tools/data-migration/faq/','/docs/v3.0/reference/tools/data-migration/faq/','/docs/v2.1/reference/tools/data-migration/faq/']
---

# TiDB Data Migration FAQ

This document collects the frequently asked questions (FAQs) about TiDB Data Migration (DM).

## Does DM support migrating data from Alibaba RDS or other cloud databases?

Currently, DM only supports decoding the standard version of MySQL or MariaDB binlog. It has not been tested for Alibaba Cloud RDS or other cloud databases. If you are confirmed that its binlog is in standard format, then it is supported.

<<<<<<< HEAD
=======
It is a known issue that for an upstream table with no primary key in Alibaba Cloud RDS, its binlog still contains a hidden primary key column, which is inconsistent with the original table structure.

Here are some known incompatible issues:

- In **Alibaba Cloud RDS**, for an upstream table with no primary key, its binlog still contains a hidden primary key column, which is inconsistent with the original table structure.
- In **HUAWEI Cloud RDS**, directly reading binlog files is not supported. For more details, see [Can HUAWEI Cloud RDS Directly Read Binlog Backup Files?](https://support.huaweicloud.com/en-us/rds_faq/rds_faq_0210.html)

>>>>>>> e5e4d8f... en: add 2 faqs and update 1 faq (#540)
## Does the regular expression of the block and allow list in the task configuration support `non-capturing (?!)`?

Currently, DM does not support it and only supports the regular expressions of the Golang standard library. See regular expressions supported by Golang via [re2-syntax](https://github.com/google/re2/wiki/Syntax).

## If a statement executed upstream contains multiple DDL operations, does DM support such migration?

DM will attempt to split a single statement containing multiple DDL change operations into multiple statements containing only one DDL operation, but might not cover all cases. It is recommended to include only one DDL operation in a statement executed upstream, or verify it in the test environment. If it is not supported, you can file an [issue](https://github.com/pingcap/dm/issues) to the DM repository.

## How to handle incompatible DDL statements?

When you encounter a DDL statement unsupported by TiDB, you need to manually handle it using dmctl (skipping the DDL statement or replacing the DDL statement with a specified DDL statement). For details, see [Skip or replace abnormal SQL statements](skip-or-replace-abnormal-sql-statements.md).

> **Note:**
>
> Currently, TiDB is not compatible with all the DDL statements that MySQL supports. See [MySQL Compatibility](https://pingcap.com/docs/dev/reference/mysql-compatibility/#ddl).

## How to reset the data migration task?

### Reset the data migration task when the relay log is in the normal state

If the relay log required by the data migration task is normal, you can use the following steps to reset the data migration task:

1. Use `stop-task` to stop abnormal data migration tasks.

2. Clean up the downstream migrated data. 

3. Choose one of the following methods to restart the data migration task:

    - Modify the task configuration file to specify a new task name, and then use `start-task` to restart the migration task. 
    - Modify the task configuration file to set `remove-meta` to `true`, and then use `start-task` to restart the migration task.

### Reset the data migration task when the relay log is in the abnormal state

#### The required relay log exists in upstream MySQL

If the relay log required by the migration task is abnormal in the DM-worker, but is normal in the upstream MySQL, you can use the following steps to restore the data migration task:

1. Use the `stop-task` command to stop all the migration tasks that are currently running. 

2. Refer to [restart DM-worker](cluster-operations.md#restart-dm-worker) to **stop** the abnormal DM-worker node.

3. Copy the normal binlog file from the upstream MySQL to replace the corresponding file in the [relay log directory](relay-log.md#directory-structure) of DM-worker.

    - If the cluster is deployed using DM-Ansible, the relay log is in the `<deploy_dir>/relay_log` directory.
    - If the cluster is manually deployed using the binary, the relay log is in the directory set by the `relay-dir` parameter.

4. Modify the information of `relay.meta` in the relay log directory of DM-worker to the information corresponding to the next binlog file.

    - If `enable-gtid` is not enabled, set `binlog-name` to the file name of the next binlog file, and set `binlog-pos` to `4`. If you copy `mysq-bin.000100` from the upstream MySQL to the relay directory, and want to continue to pull binlog from `mysql-bin.000101` later, set `binlog-name` to `mysql-bin.000101`. 

    - If `enable-gtid` is enabled, set `binlog-gtid` to the value corresponding to `Previous_gtids` at the beginning of the next binlog file. You can obtain the value by executing [SHOW BINLOG EVENTS](https://dev.mysql.com/doc/refman/5.7/en/show-binlog-events.html).

5. Refer to [restart DM-worker](cluster-operations.md#restart-dm-worker) to **start** the abnormal DM-worker node.

6. Use `start-task` to resume all stopped migration tasks.

#### The required relay log has been purged in upstream MySQL

If the relay log required by the migration task is abnormal in the DM-worker, and has been purged in the upstream MySQL, you can use the following steps to reset the data migration task:

1. Use the `stop-task` command to stop all the migration tasks that are currently running.

2. Use DM-Ansible to [stop the entire DM cluster](deploy-a-dm-cluster-using-ansible.md#step-10-stop-the-dm-cluster).

3. Manually clean up the relay log directory of the DM-worker corresponding to the MySQL cluster whose binlog is reset.

    - If the cluster is deployed using DM-Ansible, the relay log is in the `<deploy_dir>/relay_log` directory.
    - If the cluster is manually deployed using the binary, the relay log is in the directory set of the `relay-dir` parameter.

4. Clean up downstream migrated data.

5. Use DM-Ansible to [start the entire DM cluster](deploy-a-dm-cluster-using-ansible.md#step-9-deploy-the-dm-cluster).

6. Choose one of the following methods to restart the data migration task:

    - Modify the task configuration file to specify a new task name, and then use `start-task` to restart the migration task. 
    - Modify the task configuration file to set `remove-meta` to `true`, and then use `start-task` to restart the migration task.

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
<<<<<<< HEAD
=======

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

## Why does TiUP fail to deploy some versions of DM (for example, v2.0.0-hotfix)？

You can use the `tiup list dm-master` command to view the DM versions that TiUP supports to deploy. TiUP does not manage DM versions which are not shown by this command.

## How to handle the error `parse mydumper metadata error: EOF` that occurs when DM is replicating data？

You need to check the error message and log files to further analyze this error. The cause might be that the dump unit does not produce the correct metadata file due to a lack of permissions.

## Why does DM report no fatal error when replicating sharded schemas and tables, but downstream data is lost?

Check the configuration items `block-allow-list` and `table-route`:

- You need to configure the names of upstream databases and tables under `block-allow-list`. You can add "~" before `do-tables` to use regular expressions to match names.
- `table-route` uses wildcard characters instead of regular expressions to match table names. For example, `table_parttern_[0-63]` only matches 7 tables, from `table_parttern_0` to `table_pattern_6`.

## Why does the `replicate lag` monitor metric show no data when DM is not replicating from upstream?

In DM 1.0, you need to enable `enable-heartbeat` to generate the monitor data. In DM 2.0, it is expected to have no data in the monitor metric `replicate lag` because this feature is not supported.

## How to handle the error `fail to initial unit Sync of subtask` when DM is starting a task, with the `RawCause` in the error message showing `context deadline exceeded`?

This is a known issue in DM 2.0.0 version and will be fixed in DM 2.0.1 version. It is likely to be triggered when a replication task has a lot of tables to process. If you use TiUP to deploy DM, you can upgrade DM to the nightly version to fix this issue. Or you can download the 2.0.0-hotfix version from [the release page of DM](https://github.com/pingcap/dm/releases) on GitHub and manually replace the executable files.

## How to handle the error `duplicate entry` when DM is replicating data?

You need to first check and confirm the following things:

- `disable-detect` is not configured in the replication task.
- The data is not inserted manually or by other replication programs.
- No DML filter associated with this table is configured.

To facilitate troubleshooting, you can first collect general log files of the downstream TiDB instance and then ask for technical support at [TiDB Community slack channel](https://tidbcommunity.slack.com/archives/CH7TTLL7P). The following example shows how to collect general log files:

```bash
# Enable general log collection
curl -X POST -d "tidb_general_log=1" http://{TiDBIP}:10080/settings
# Disable general log collection
curl -X POST -d "tidb_general_log=0" http://{TiDBIP}:10080/settings
```

When the `duplicate entry` error occurs, you need to check the log files for the records that contain conflict data.

## Why do some monitoring panels show `No data point`?

It is normal for some panels to have no data. For example, when there is no error reported, no DDL lock, or the relay log feature is not enabled, the corresponding panels show `No data point`. For detailed description of each panel, see [DM Monitoring Metrics](monitor-a-dm-cluster.md).

## In DM v1.0, why does the command `sql-skip` fail to skip some statements when the task is in error?

You need to first check whether the binlog position is still advancing after you execute `sql-skip`. If so, it means that `sql-skip` has taken effect. The reason why this error keeps occurring is that the upstream sends multiple unsupported DDL statements. You can use `sql-skip -s <sql-pattern>` to set a pattern to match these statements.

Sometimes, the error message contains the `parse statement` information, for example:

```
if the DDL is not needed, you can use a filter rule with \"*\" schema-pattern to ignore it.\n\t : parse statement: line 1 column 11 near \"EVENT `event_del_big_table` \r\nDISABLE\" %!!(MISSING)(EXTRA string=ALTER EVENT `event_del_big_table` \r\nDISABLE
```

The reason for this type of error is that the TiDB parser cannot parse DDL statements sent by the upstream, such as `ALTER EVENT`, so `sql-skip` does not take effect as expected. You can add [binlog event filters](key-features.md#binlog-event-filter) in the configuration file to filter those statements and set `schema-pattern: "*"`. Starting from DM v2.0.1, DM pre-filters statements related to `EVENT`.

In DM v2.0, `handle-error` replaces `sql-skip`. You can use `handle-error` instead to avoid this issue.

## Why do `REPLACE` statements keep appearing in the downstream when DM is replicating?

You need to check whether the [safe mode](glossary.md#safe-mode) is automatically enabled for the task. If the task is automatically resumed after an error, or if there is high availability scheduling, then the safe mode is enabled because it is within 5 minutes after the task is started or resumed.

You can check the DM-worker log file and search for a line containing `change count`. If the `new count` in the line is not zero, the safe mode is enabled. To find out why it is enabled, check when it happens and if any errors are reported before.
>>>>>>> e5e4d8f... en: add 2 faqs and update 1 faq (#540)
