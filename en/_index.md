---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
---

# TiDB Data Migration Documentation

[DM](https://github.com/pingcap/dm) (Data Migration) is an integrated data replication task management platform that supports the full data migration and the incremental data migration from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

## Architecture

The Data Migration tool includes three components: DM-master, DM-worker, and dmctl.

![Data Migration architecture](/media/dm-architecture.png)

### DM-master

DM-master manages and schedules the operation of data replication tasks.

- Storing the topology information of the DM cluster
- Monitoring the running state of DM-worker processes
- Monitoring the running state of data replication tasks
- Providing a unified portal for the management of data replication tasks
- Coordinating the DDL replication of sharded tables in each instance under the sharding scenario

### DM-worker

DM-worker executes specific data replication tasks.

- Persisting the binlog data to the local storage
- Storing the configuration information of the data replication subtasks
- Orchestrating the operation of the data replication subtasks
- Monitoring the running state of the data replication subtasks

After DM-worker is started, it automatically replicates the upstream binlog to the local configuration directory (the default replication directory is `<deploy_dir>/relay_log` if DM is deployed using `DM-Ansible`). For details about DM-worker, see [DM-worker Introduction](dm-worker-intro.md). For details about the relay log, see [Relay Log](relay-log.md).

### dmctl

dmctl is the command line tool used to control the DM cluster.

- Creating/Updating/Dropping data replication tasks
- Checking the state of data replication tasks
- Handling the errors during data replication tasks
- Verifying the configuration correctness of data replication tasks

## Data replication features

This section describes the data replication features provided by the Data Migration tool.

### Schema and table routing

The schema and table routing feature means that DM can replicate a certain table of the upstream MySQL or MariaDB instance to the specified table in the downstream, which can be used to merge or replicate the sharding data.

### Black and white lists replication at the schema and table levels

The black and white lists filtering rule of the upstream database instance tables is similar to MySQL `replication-rules-db`/`replication-rules-table`, which can be used to filter or only replicate all operations of some databases or some tables.

### Binlog event filtering

Binlog event filtering is a more fine-grained filtering rule than the black and white lists filtering rule. You can use statements like `INSERT` or `TRUNCATE TABLE` to specify the binlog events of `schema/table` that you need to replicate or filter out.

### Sharding support

DM supports merging the original sharded instances and tables into TiDB, but with some restrictions.

## Usage restrictions

Before using the DM tool, note the following restrictions:

+ Database version
+ DDL syntax
+ Sharding
+ Operations
+ Switching DM-worker connection to another MySQL instance
