---
title: Migration When the Downstream Table Has More Columns
summary: Learn how to use DM to migrate tables when the downstream table schema has more columns.
---

# Migration When the Downstream Table Has More Columns

This document describes how to migrate tables using DM when the downstream table schema has more columns than the upstream table schema.

> **Note:**
>
> * The columns that only exist in the downstream must be given a default value or allowed to be `NULL`.
> * For tables that are being migrated by DM, you can directly add new columns in the downstream that are given a default value or allowed to be `NULL`. Adding such new columns does not affect the data migration.
> * The columns that only exist in the downstream have default values or are allowed to be `NULL`, there is no need for special configuration and processing for full export and import.

## Migration solution

Assume the upstream table schema before the data migration is as follows:

```sql
CREATE TABLE `t1` (
  `c1` int(11) NOT NULL,
  `c2` int(11) DEFAULT NULL,
  PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

The downstream table schema is as follows:

```sql
CREATE TABLE `t1` (
  `c1` int(11) NOT NULL,
  `c2` int(11) DEFAULT NULL,
  `c3` varchar(256) DEFAULT 'default_c3',
  PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin
```

When you replicate incremental data using DM, if the downstream table schema has more columns than the upstream table schema, you can see the similar error by running the `query-status` command:

```
"errors": [
    {
        "ErrCode": 36027,
        "ErrClass": "sync-unit",
        "ErrScope": "internal",
        "ErrLevel": "high",
        "Message": "startLocation: [position: (mysql-bin.000001, 17491), gtid-set: ], endLocation: [position: (mysql-bin.000001, 17535), gtid-set: ]: gen insert sqls failed, schema: db_single, table: t1: Column count doesn't match value count: 3 (columns) vs 2 (values)",
        "RawCause": "",
        "Workaround": ""
    }
]
```

When a DML statement in the binlog event needs to be migrated, if DM has not maintained internally the table schema corresponding to that table, DM tries to use the current table schema in the downstream to parse the binlog event and generate the corresponding DML statement. If the number of columns in the binlog event is inconsistent with the number of columns in the downstream table schema, the above error might occur.

In such cases, you can run the [`operate-schema`](manage-schema.md) command to specify for the table a table schema that matches the binlog event. The steps are as follows:

1. Prepare the `CREATE TABLE` table schema statement that corresponds to data in the binlog event and save this statement in a file. For example, save the following table schema in the `db_single.t1-schema.sql` file:

    ```sql
    CREATE TABLE `t1` (
      `c1` int(11) NOT NULL,
      `c2` int(11) DEFAULT NULL,
      PRIMARY KEY (`c1`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1
    ```

2. Execute the [`operate-schema`](manage-schema.md) command to set the table schema. At this time, the task should be in the `Paused` state because of the above error.

    {{< copyable "shell-regular" >}}

    ```bash
    operate-schema set -s mysql-replica-01 task_single -d db_single -t t1 db_single.t1-schema.sql
    ```    

3. Execute the [`resume-task`](resume-task.md) command to resume the `Paused` task.

4. Execute the [`query-status`](query-status.md) command to check whether the data migration task is running normally.
