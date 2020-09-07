---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
aliases: ['/docs/tidb-data-migration/','/docs/tidb-data-migration/stable/','/docs/tidb-data-migration/v1.0/']
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform that supports the full data migration and the incremental data replication from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

> **Note:**
>
> DM replicates data to TiDB in the form of SQL statements, so each version of DM is compatible with **all versions** of TiDB. In the production environment, it is recommended to use the latest released version of DM. To install DM, see [DM download link](https://pingcap.com/docs/stable/reference/tools/download/#tidb-dm-data-migration).

<<<<<<< HEAD
## Architecture
=======
- High availability. The data migration task can run normally even when some DM-master or DM-worker nodes fail.
- [Sharding DDL support in the optimistic mode](feature-shard-merge-optimistic.md). In this mode, migration latency can be reduced in some scenarios and you can make A/B changes in the upstream database.
- Better usability, including the new [error handling mechanism](handle-failed-sql-statements.md) and the easier-to-read error messages and error handling suggestions.
- [TLS support](enable-tls.md) for connections between the upstream and the downstream, and for connections between DM components.
- Support for migrating data from MySQL 8.0 (experimental).
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

The Data Migration tool includes three components: DM-master, DM-worker, and dmctl.

![Data Migration architecture](/media/dm-architecture.png)

### DM-master

DM-master manages and schedules the operation of data replication tasks.

<<<<<<< HEAD
- Storing the topology information of the DM cluster
- Monitoring the running state of DM-worker processes
- Monitoring the running state of data replication tasks
- Providing a unified portal for the management of data replication tasks
- Coordinating the DDL replication of sharded tables in each instance under the sharding scenario
=======
- [Usage Scenarios](usage-scenario-shard-merge.md)
- [Quick Start with DM](quick-start-with-dm.md)
- [Migrate Data Using DM](migrate-data-using-dm.md)
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

### DM-worker

DM-worker executes specific data replication tasks.

<<<<<<< HEAD
- Persisting the binlog data to the local storage
- Storing the configuration information of the data replication subtasks
- Orchestrating the operation of the data replication subtasks
- Monitoring the running state of the data replication subtasks
=======
- [Deploy DM Using TiUP](deploy-a-dm-cluster-using-tiup.md)
- [Deploy DM Using Binary](deploy-a-dm-cluster-using-binary.md)
- [Use DM to Migrate Data](migrate-data-using-dm.md)
- [Monitor and Alert](monitor-a-dm-cluster.md)
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

After DM-worker is started, it automatically replicates the upstream binlog to the local configuration directory (the default replication directory is `<deploy_dir>/relay_log` if DM is deployed using `DM-Ansible`). For details about DM-worker, see [DM-worker Introduction](dm-worker-intro.md). For details about the relay log, see [Relay Log](relay-log.md).

### dmctl

dmctl is the command line tool used to control the DM cluster.

- Creating/Updating/Dropping data replication tasks
- Checking the state of data replication tasks
- Handling the errors during data replication tasks
- Verifying the configuration correctness of data replication tasks

## Data replication features

<<<<<<< HEAD
This section describes the data replication features provided by the Data Migration tool.
=======
- [Simple Usage Scenario](usage-scenario-simple-migration.md)
- [Shard Merge Scenario](usage-scenario-shard-merge.md)
- [Migrate from MySQL (Amazon Aurora)](migrate-from-mysql-aurora.md)
- [Shard Merge Scenario Best Practices](shard-merge-best-practices.md)
- [Switch DM-worker connection](usage-scenario-master-slave-switch.md)
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

### Schema and table routing

The schema and table routing feature means that DM can replicate a certain table of the upstream MySQL or MariaDB instance to the specified table in the downstream, which can be used to merge or replicate the sharding data.

### Block and allow lists replication at the schema and table levels

The block and allow lists filtering rule of the upstream database instance tables is similar to MySQL `replication-rules-db`/`replication-rules-table`, which can be used to filter or only replicate all operations of some databases or some tables.

### Binlog event filtering

Binlog event filtering is a more fine-grained filtering rule than the block and allow lists filtering rule. You can use statements like `INSERT` or `TRUNCATE TABLE` to specify the binlog events of `schema/table` that you need to replicate or filter out.

### Sharding support

DM supports merging the original sharded instances and tables into TiDB, but with some restrictions.

## Usage restrictions

Before using the DM tool, note the following restrictions:

+ Database version
+ DDL syntax
+ Sharding
+ Operations
+ Switching DM-worker connection to another MySQL instance
