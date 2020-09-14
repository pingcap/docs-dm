---
title: Data Migration Overview
summary: Learn about the Data Migration tool, the architecture, the key components, and features.
---

<!-- markdownlint-disable MD007 -->

# Data Migration Overview

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform that supports the full data migration and the incremental data replication from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

**What's new in DM v2.0:**

- [High availability of data migration task](#high-availability). The data migration task can run normally even when some DM-master or DM-worker nodes fail.
- [Sharding DDL support in the optimistic mode](feature-shard-merge-optimistic.md). In this mode, migration latency can be reduced in some scenarios and you can make A/B changes in the upstream database.
- Better usability, including the new [error handling mechanism](handle-failed-sql-statements.md) and the easier-to-read error messages and error handling suggestions.
- [TLS support](enable-tls.md) for connections between the upstream and the downstream, and for connections between DM components.
- Support for migrating data from MySQL 8.0 (experimental).

> **Note:**
>
> DM migrates data to TiDB in the form of SQL statements, so each version of DM is compatible with **all versions** of TiDB. In the production environment, it is recommended to use the latest released version of DM. To install DM, see [DM download link](https://pingcap.com/docs/stable/reference/tools/download/#tidb-dm-data-migration).

## Architecture

The Data Migration tool includes three components: DM-master, DM-worker, and dmctl.

![Data Migration architecture](/media/dm-architecture-2.0.png)

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

For more details of DM-worker, see [DM-worker Introduction](dm-worker-intro.md).

### dmctl

dmctl is the command line tool used to control the DM cluster.

- Creating/Updating/Dropping data migration tasks
- Checking the state of data migration tasks
- Handling the errors during data migration tasks
- Verifying the configuration correctness of data migration tasks

### High availability

When you deploy multiple DM-master nodes, all DM-master nodes use the embedded etcd to form a cluster. The DM-master cluster is used to store metadata such as cluster node information and task configuration. The leader node elected through etcd is used to provide services such as cluster management and data migration task management. Therefore, if the number of available DM-master nodes exceeds half of the deployed nodes, the DM cluster can normally provide services.

When the number of deployed DM-worker nodes exceeds the number of upstream MySQL/MariaDB nodes, the extra DM-worker nodes are idle by default. If a DM-worker node goes offline or is isolated from the DM-master leader, DM-master automatically schedules data migration tasks of the original DM-worker node to other idle DM-worker nodes. (If a DM-worker node is isolated, it automatically stops the data migration tasks on it); if there are no available idle DM-worker nodes, the data migration tasks of the original DM-worker cannot be performed.

## Data migration features

This section describes the data migration features provided by the Data Migration tool.

### Schema and table routing

The [schema and table routing](key-features.md#table-routing) feature means that DM can migrate a certain table of the upstream MySQL or MariaDB instance to the specified table in the downstream, which can be used to merge or migrate the sharding data.

### Block and allow lists migration at the schema and table levels

The [block and allow lists filtering rule](key-features.md#block-and-allow-table-lists) of the upstream database instance tables is similar to MySQL `replication-rules-db`/`replication-rules-table`, which can be used to filter or only replicate all operations of some databases or some tables.

### Binlog event filtering

[Binlog event filtering](key-features.md#binlog-event-filter) is a more fine-grained filtering rule than the block and allow lists filtering rule. You can use statements like `INSERT` or `TRUNCATE TABLE` to specify the binlog events of `schema/table` that you need to migrate or filter out.

### Sharding support

DM supports merging the original sharded instances and tables into TiDB, but with some restrictions. For details, see [Sharding DDL usage Restrictions in the pessimistic mode](feature-shard-merge-pessimistic.md#restrictions) and [Sharding DDL usage Restrictions in the optimistic mode](feature-shard-merge-optimistic.md#restrictions).

## Usage restrictions

Before using the DM tool, note the following restrictions:

+ Database version

    - 5.5 < MySQL version < 8.0
    - MariaDB version >= 10.1.2

    > **Note:**
    >
    > If there is a primary-secondary migration structure between the upstream MySQL/MariaDB servers, then choose the following version.
    >
    > - 5.7.1 < MySQL version < 8.0
    > - MariaDB version >= 10.1.3

    Data Migration [prechecks the corresponding privileges and configuration automatically](precheck.md) while starting the data migration task using dmctl.

+ DDL syntax

    - Currently, TiDB is not compatible with all the DDL statements that MySQL supports. Because DM uses the TiDB parser to process DDL statements, it only supports the DDL syntax supported by the TiDB parser. For details, see [MySQL Compatibility](https://pingcap.com/docs/stable/reference/mysql-compatibility/#ddl).

    - DM reports an error when it encounters an incompatible DDL statement. To solve this error, you need to manually handle it using dmctl, either skipping this DDL statement or replacing it with a specified DDL statement(s). For details, see [Skip or replace abnormal SQL statements](faq.md#how-to-handle-incompatible-ddl-statements).

+ Sharding

    - If conflict exists between sharded tables, solve the conflict by referring to [handling conflicts of auto-increment primary key](shard-merge-best-practices.md#handle-conflicts-of-auto-increment-primary-key). Otherwise, data migration is not supported. Conflicting data can cover each other and cause data loss.

    - For other sharding restrictions, see [Sharding DDL usage Restrictions in the pessimistic mode](feature-shard-merge-pessimistic.md#restrictions) and [Sharding DDL usage Restrictions in the optimistic mode](feature-shard-merge-optimistic.md#restrictions).

+ Operations

    - After DM-worker is restarted, the DDL lock migration cannot be automatically restored in some conditions. You need to manually handle it. For details, see [Handle Sharding DDL Locks Manually](manually-handling-sharding-ddl-locks.md).

+ Switching DM-worker connection to another MySQL instance

    When DM-worker connects the upstream MySQL instance via a virtual IP (VIP), if you switch the VIP connection to another MySQL instance, DM might connect to the new and old MySQL instances at the same time in different connections. In this situation, the binlog migrated to DM is not consistent with other upstream status that DM receives, causing unpredictable anomalies and even data damage. To make necessary changes to DM manually, see [Switch DM-worker connection via virtual IP](usage-scenario-master-slave-switch.md#switch-dm-worker-connection-via-virtual-ip).
