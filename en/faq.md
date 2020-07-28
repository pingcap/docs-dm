---
title: TiDB Data Migration FAQ
summary: Learn about frequently asked questions (FAQs) about TiDB Data Migration (DM).
aliases: ['/docs/tidb-data-migration/dev/faq/']
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

When an exception occurs during data migration and the data migration task cannot be resumed, you need to reset the task and re-migrate the data:

1. Execute the `stop-task` command to stop the abnormal data migration task.

2. Purge the data replicated to the downstream.

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
    
- DM [processes the gh-ost table during the `alter ghost_table` operation](online-ddl-scheme.md#online-schema-change-gh-ost) and records the DDL information of `ghost_table`;
- When DM-worker is restarted to start the task, DM reads the DDL from `dm_meta.{task_name}_onlineddl`.

Therefore, in the process of incremental replication, if the specified Pos has skipped the `alter ghost_table` DDL but the Pos is still in the online-ddl process of gh-ost, the ghost_table is not written into memory or `dm_meta.{task_name}_onlineddl` correctly. In such cases, the above error is returned.

You can avoid this error by the following steps:

1. Remove the `online-ddl-scheme` configuration of the task.

2. Configure `_{table_name}_gho`, `_{table_name}_ghc`, and `_{table_name}_del` in `block-allow-list.ignore-tables`.

3. Execute the upstream DDL in the downstream TiDB manually.

4. After the Pos is replicated to the position after the gh-ost process, re-enable the `online-ddl-scheme` and comment out `block-allow-list.ignore-tables`.
