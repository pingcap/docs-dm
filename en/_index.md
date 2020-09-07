---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
aliases: ['/docs/tidb-data-migration/','/docs/tidb-data-migration/stable/','/docs/tidb-data-migration/v1.0/']
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform that supports the full data migration and the incremental data replication from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

> **Note:**
>
> DM migrates data to TiDB in the form of SQL statements, so each version of DM is compatible with **all versions** of TiDB. In the production environment, it is recommended to use the latest released version of DM. To install DM, see [DM download link](https://pingcap.com/docs/stable/reference/tools/download/#tidb-dm-data-migration).

## Architecture

The Data Migration tool includes three components: DM-master, DM-worker, and dmctl.

![Data Migration architecture](/media/dm-architecture.png)

### DM-master

DM-master manages and schedules the operation of data migration tasks.

- Storing the topology information of the DM cluster
- Monitoring the running state of DM-worker processes
- Monitoring the running state of data migration tasks
- Providing a unified portal for the management of data migration tasks
- Coordinating the DDL migration of sharded tables in each instance under the sharding scenario

### DM-worker

DM-worker executes specific data migration tasks.

- Persisting the binlog data to the local storage
- Storing the configuration information of the data migration subtasks
- Orchestrating the operation of the data migration subtasks
- Monitoring the running state of the data migration subtasks

After DM-worker is started, it automatically migrates the upstream binlog to the local configuration directory (the default migration directory is `<deploy_dir>/relay_log` if DM is deployed using `DM-Ansible`). For details about DM-worker, see [DM-worker Introduction](dm-worker-intro.md). For details about the relay log, see [Relay Log](relay-log.md).

### dmctl

dmctl is the command line tool used to control the DM cluster.

- Creating/Updating/Dropping data migration tasks
- Checking the state of data migration tasks
- Handling the errors during data migration tasks
- Verifying the configuration correctness of data migration tasks

## Data migration features

This section describes the data migration features provided by the Data Migration tool.

### Schema and table routing

The schema and table routing feature means that DM can migrate a certain table of the upstream MySQL or MariaDB instance to the specified table in the downstream, which can be used to merge or migrate the sharding data.

### Block and allow lists migration at the schema and table levels

The block and allow lists filtering rule of the upstream database instance tables is similar to MySQL `migration-rules-db`/`migration-rules-table`, which can be used to filter or only migrate all operations of some databases or some tables.

### Binlog event filtering

Binlog event filtering is a more fine-grained filtering rule than the block and allow lists filtering rule. You can use statements like `INSERT` or `TRUNCATE TABLE` to specify the binlog events of `schema/table` that you need to migrate or filter out.

### Sharding support

DM supports merging the original sharded instances and tables into TiDB, but with some restrictions.

## Usage restrictions

Before using the DM tool, note the following restrictions:

+ Database version
+ DDL syntax
+ Sharding
+ Operations
+ Switching DM-worker connection to another MySQL instance
