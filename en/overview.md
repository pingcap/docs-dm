---
title: Data Migration Overview
summary: Learn about the Data Migration tool, the architecture, the key components, and features.
aliases: ['/docs/tidb-data-migration/dev/overview/']
---

<!-- markdownlint-disable MD007 -->

# Data Migration Overview

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform, which supports the full data migration and the incremental data replication from MySQL-compatible databases (such as MySQL, MariaDB, and Aurora MySQL) into TiDB. It can help to reduce the operation cost of data migration and simplify the troubleshooting process.

## Basic features

This section describes the basic data migration features provided by DM.

![DM Core Features](/media/dm-core-features.png)

### Block and allow lists migration at the schema and table levels

The [block and allow lists filtering rule](key-features.md#block-and-allow-table-lists) is similar to the `replication-rules-db`/`replication-rules-table` feature of MySQL, which can be used to filter or only replicate all operations of some databases or some tables.

### Binlog event filtering

The [binlog event filtering](key-features.md#binlog-event-filter) feature means that DM can filter certain types of SQL statements from certain tables in the source database. For example, you can filter all `INSERT` statements in the table `test`.`sbtest` or filter all `TRUNCATE TABLE` statements in the schema `test`.

### Schema and table routing

The [schema and table routing](key-features.md#table-routing) feature means that DM can migrate a certain table of the source database to the specified table in the downstream. For example, you can migrate the data from the table `test`.`sbtest1` in the source database to the table `test`.`sbtest2` in TiDB. This is also a core feature for merging and migrating sharded databases and tables.

## Advanced features

### Shard merge and migration

DM supports merging and migrating the original sharded instances and tables from the source databases into TiDB, but with some restrictions. For details, see [Sharding DDL usage restrictions in the pessimistic mode](feature-shard-merge-pessimistic.md#restrictions) and [Sharding DDL usage restrictions in the optimistic mode](feature-shard-merge-optimistic.md#restrictions).

### Optimization for third-party online-schema-change tools in the migration process

In the MySQL ecosystem, tools such as gh-ost and pt-osc are widely used. DM provides support for these tools to avoid migrating unnecessary intermediate data. For details, see [Online DDL Tools](key-features.md#online-ddl-tools)

## Usage restrictions

Before using the DM tool, note the following restrictions:

+ Database version

    - MySQL version > 5.5
    - MariaDB version >= 10.1.2

    > **Note:**
    >
    > If there is a primary-secondary migration structure between the upstream MySQL/MariaDB servers, then choose the following version.
    >
    > - MySQL version > 5.7.1
    > - MariaDB version >= 10.1.3

    > **Warning:**
    >
    > Support for MySQL 8.0 is an experimental feature of TiDB Data Migration v2.0. It is **NOT** recommended that you use it in a production environment.

+ DDL syntax compatibility

    - Currently, TiDB is not compatible with all the DDL statements that MySQL supports. Because DM uses the TiDB parser to process DDL statements, it only supports the DDL syntax supported by the TiDB parser. For details, see [MySQL Compatibility](https://pingcap.com/docs/stable/reference/mysql-compatibility/#ddl).

    - DM reports an error when it encounters an incompatible DDL statement. To solve this error, you need to manually handle it using dmctl, either skipping this DDL statement or replacing it with a specified DDL statement(s). For details, see [Skip or replace abnormal SQL statements](faq.md#how-to-handle-incompatible-ddl-statements).

+ Sharding

    - If conflict exists between sharded tables, solve the conflict by referring to [handling conflicts of auto-increment primary key](shard-merge-best-practices.md#handle-conflicts-of-auto-increment-primary-key). Otherwise, data migration is not supported. Conflicting data can cover each other and cause data loss.

    - For other sharding DDL migration restrictions, see [Sharding DDL usage restrictions in the pessimistic mode](feature-shard-merge-pessimistic.md#restrictions) and [Sharding DDL usage restrictions in the optimistic mode](feature-shard-merge-optimistic.md#restrictions).

+ Consistent connection switch of MySQL instances

    When DM-worker connects the upstream MySQL instance via a virtual IP (VIP), if you switch the VIP connection to another MySQL instance, DM might connect to the new and old MySQL instances at the same time in different connections. In this situation, the binlog migrated to DM is not consistent with other upstream status that DM receives, causing unpredictable anomalies and even data damage. To make necessary changes to DM manually, see [Switch DM-worker connection via virtual IP](usage-scenario-master-slave-switch.md#switch-dm-worker-connection-via-virtual-ip).
