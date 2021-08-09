---
title: Migration when the Downstream TiDB Table Has More Columns
summary: Learn how to use Data Migration (DM) to migrate tables when the downstream table schema has more columns.
---

# Migration when the Downstream TiDB Table Has More Columns

This document describes how to migrate tables using DM when the downstream TiDB table schema has more columns than the upstream table schema.

## The table shcema of the data source

This document uses the follwing data source example:

| Schema | Tables |
|:------|:------|
| user  | information, log |
| store | store_bj, store_tj |
| log   | messages |

## Migration requirements

Customize a table `log.messages` in TiDB, whose table shema has all the columns in the `log.messages` table of data source, and has more columns than the data source table schema. Under this condition, migrate the table `log.messages` of data source to the table `log.messages` of TiDB cluster.

> **Note:**
>
> * The columns that only exist in the downstream TiDB must be given a default value or allowed to be `NULL`.
> * For tables that are being migrated by DM, you can directly add new columns in the downstream TiDB that are given a default value or allowed to be `NULL`. Adding such new columns does not affect the data migration.

## Migration solution

This section describes the specific steps for meeting the above migration requirements.

## Manually create a table schema downsream TiDB

Assume the data source table schema before the data migration is as follows:

```sql
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

Manually create a table schema downsream TiDB, which has an additional column `annotation` than data source:

```sql
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL,
  `annotation` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

### Create a data migration task

1. Create a task configuration file `task.yaml` configured with incremental replication mode and the replication starting point of every data source.The complete task configuration file is as follows:

   {{< copyable "yaml" >}}

   ```yaml
   name: task-test   # The name of the task. Should be globally unique.
   task-mode: all    # full data migration + incremental data migration

   ## Configure the access information of downstream TiDB database instance
   target-database:       # the configuration of downstream database instance.
     host: "127.0.0.1"
     port: 4000
     user: "root"
     password: ""         # It is recommended to use password encrypted with dmctl if the password is not empty.

   ##  Configure tables to be migrate using block-allow-list
   block-allow-list:   # The filter rule set of the block allow list of the matched table of the upstream database instance. Use black-white-list if the DM's version is earlier than v2.0.0-beta.2.
       do-dbs: ["log"] # the database to be migrated.

   ## Coinfigure data source
   mysql-instances:
     - source-id: "mysql-01"         # the ID of the data source and can be obtained from the data source configuration
       block-allow-list: "bw-rule-1" # To import the block-allow-list configuration above.
       syncer-config-name: "global"  # To import the incremental data migration configuration of syncers.
       meta:                         # If `task-mode` is `incremental` and the `checkpoint` in downstream database does not exist, `meta` is the starting point of binlog; If `checkpoint` exists, base it on `checkpoint`.
         binlog-name: "mysql-bin.000001"
         binlog-pos: 2022
         binlog-gtid: "09bec856-ba95-11ea-850a-58f2b4af5188:1-9"
   ```

2. Create a replication task using `start-task` command

   {{< copyable "shell-regular" >}}

   ```bash
   tiup dmctl --master-addr <master-addr> start-task task.yaml
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

3. Use `query-status` to make sure no error message occurs in the replication task:

   {{< copyable "shell-regular" >}}

   ```bash
   tiup dmctl --master-addr <master-addr> query-status test
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
                       "name": "task-test",
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

## Only migrate incremental data to TiDB and the downstream TiDB table has more columns

If your migration task contains full data migration, the task can operate normally. If you have already used other tools to do full data migration and this migration task only uses DM to replicate incremental data, refer to [Migrate Incremental Data to TiDB](usage-scenario-incremental-migration.md#create-sync-task) to create a data migration task. At the same time, you need to manually configure the table schema in DM for MySQL binlog parsing.

Otherwise, after creating the task, the following data migration errors occur when you run `query-status` command:

```
"errors": [
    {
        "ErrCode": 36027,
        "ErrClass": "sync-unit",
        "ErrScope": "internal",
        "ErrLevel": "high",
        "Message": "startLocation: [position: (mysql-bin.000001, 2022), gtid-set:09bec856-ba95-11ea-850a-58f2b4af5188:1-9 ], endLocation: [position: (mysql-bin.000001, 2022), gtid-set: 09bec856-ba95-11ea-850a-58f2b4af5188:1-9]: gen insert sqls failed, schema: log, table: messages: Column count doesn't match value count: 3 (columns) vs 2 (values)",
        "RawCause": "",
        "Workaround": ""
    }
]
```

The reason for the above errors is that when DM migrates the binlog event, if DM has not maintained internally the table schema corresponding to that table, DM tries to use the current table schema in the downstream to parse the binlog event and generate the corresponding DML statement. If the number of columns in the binlog event is inconsistent with the number of columns in the downstream table schema, the above error might occur.

In such cases, you can run the [`operate-schema`](manage-schema.md) command to specify for the table a table schema that matches the binlog event. If you are migrating sharded tables, you need to configure the table schema in DM for parsing MySQL binlog for each sharded tables according to the following steps:

1. Specify the table schema for the table `log.messages` to be migrated in the data source. The table schema needs to correspond to the data of the binlog event to be replicated by DM. Then save the `CREATE TABLE` table schema statement in a file. For example, save the following table schema in the `log.messages.sql` file:

    ```sql
    CREATE TABLE `messages` (
      `id` int(11) NOT NULL,
      `message` varchar(255) DEFAULT NULL,
      PRIMARY KEY (`id`)
    )
    ```

2. Execute the [`operate-schema`](manage-schema.md) command to set the table schema. At this time, the task should be in the `Paused` state because of the above error.

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr <master-addr> operate-schema set -s mysql-01 task-test -d log -t message log.message.sql
    ```    

3. Execute the [`resume-task`](resume-task.md) command to resume the `Paused` task.

4. Execute the [`query-status`](query-status.md) command to check whether the data migration task is running normally.
