---
title: TiDB Data Migration FAQ
summary: Learn about frequently asked questions (FAQs) about TiDB Data Migration (DM).
aliases: ['/docs/tidb-data-migration/stable/faq/','/docs/tidb-data-migration/v1.0/faq/','/docs/dev/faq/data-migration/','/docs/dev/reference/tools/data-migration/faq/','/docs/v3.1/reference/tools/data-migration/faq/','/docs/v3.0/reference/tools/data-migration/faq/','/docs/v2.1/reference/tools/data-migration/faq/']
---

# TiDB Data Migration FAQ

This document collects the frequently asked questions (FAQs) about TiDB Data Migration (DM).

## Does DM support replicating data from Alibaba RDS or other cloud databases?

Currently, DM only supports decoding the standard version of MySQL or MariaDB binlog. It has not been tested for Alibaba Cloud RDS or other cloud databases. If you are confirmed that its binlog is in standard format, then it is supported.

## Does the regular expression of the block and allow list in the task configuration support `non-capturing (?!)`?

Currently, DM does not support it and only supports the regular expressions of the Golang standard library. See regular expressions supported by Golang via [re2-syntax](https://github.com/google/re2/wiki/Syntax).

## If a statement executed upstream contains multiple DDL operations, does DM support such replication?

DM will attempt to split a single statement containing multiple DDL change operations into multiple statements containing only one DDL operation, but might not cover all cases. It is recommended to include only one DDL operation in a statement executed upstream, or verify it in the test environment. If it is not supported, you can file an [issue](https://github.com/pingcap/dm/issues) to the DM repository.

## How to handle incompatible DDL statements?

When you encounter a DDL statement unsupported by TiDB, you need to manually handle it using dmctl (skipping the DDL statement or replacing the DDL statement with a specified DDL statement). For details, see [Skip or replace abnormal SQL statements](skip-or-replace-abnormal-sql-statements.md).

> **Note:**
>
> Currently, TiDB is not compatible with all the DDL statements that MySQL supports. See [MySQL Compatibility](https://pingcap.com/docs/dev/reference/mysql-compatibility/#ddl).

## How to reset the data migration task?

### Reset the data migration task when the relay log is in the normal state

If the relay log required by the data migration task is normal, you can use the following steps to restore the data migration task:

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
    - If the cluster is manually deployed using the binary, the relay log is in the directory set of the `relay-dir` parameter.

4. Modify the information of `relay.meta` in the relay log directory of DM-worker to the information corresponding to the next binlog file.

    - If `enable-gtid` is not enabled, set `binlog-name` to the file name of the next binlog file, and set `binlog-pos` to `4`. If you copy `mysq-bin.000100` from the upstream MySQL to the relay directory, and want to continue to pull binlog from `mysql-bin.000101` later, set `binlog-name` to `mysql-bin.000101` `. 

    - If `enable-gtid` is enabled, set `binlog-gtid` to the value corresponding to `Previous_gtids` at the beginning of the next binlog file. You can obtain the value by executing [SHOW BINLOG EVENTS](https:// dev.mysql.com/doc/refman/5.7/en/show-binlog-events.html).

5. Refer to [restart DM-worker](cluster-operations.md#restart-dm-worker) to **start** the abnormal DM-worker node.

6. Use `start-task` to resume a stopped migration task.

#### The required relay log has been cleaned up in upstream MySQL

If the relay log required by the migration task is abnormal in the DM-worker, and has been cleaned up in the upstream MySQL, you can use the following steps to restore the data migration task:

1. Use the `stop-task` command to stop all the migration tasks that are currently running.

2. Use DM-Ansible to [stop the entire DM cluster](deploy-a-dm-cluster-using-ansible.md#step-10-stop-the-dm-cluster).

3. Manually clean up the relay log directory of the DM-worker corresponding to the MySQL master whose binlog is reset.

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
