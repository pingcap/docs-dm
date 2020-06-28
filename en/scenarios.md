---
title: Usage scenario
summary: Understand the main usage scenarios supported by TiDB Data Migration.
category: reference
---

# Usage scenarios

This document introduces the main usage scenarios supported by TiDB Data Migration (DM) and related usage suggestions.

## Non-shard-merge scenario

### TiDB as MySQL/MariaDB's secondary library

If you want to use TiDB as the upstream MySQL/MariaDB's secondary library, that is, import all the data in the upstream instance to TiDB in full mode, and then replicate the changes data to TiDB in the incremental mode in real-time, simply configure the data migration task according to the following rules:

-Specify `task-mode` as `all`.
-Configure `target-database` as downstream TiDB's connection information.
-Configure the `source-id` for upstream MySQL/MariaDB in `mysql-instances`.
-Use default concurrency control parameters or configure `mydumper-thread`, `loader-thread`, and `syncer-thread` as needed.
-No need to configure `route-rules`, `filter-rules` and `black-white-list` etc.

### Migrating some business data in MySQL/MariaDB

If there are multiple business data in the original MySQL/MariaDB, but only part of the data needs to be migrated to TiDB, you can configure the migration task according to [Use TiDB as a secondary library of MySQL/MariaDB](#tidb-as-mysql/mariadb's-secondary-library), and then configure `black-white-list` as needed.

If you want to migrate upstream data to a schema or table with a different name in downstream, you can additionally configure `route-rules`.

For some archiving scenarios, some data may be periodically cleaned up through `TRUNCATE TABLE`/`DROP TABLE` or other methods in the upstream, but it is expected that all data will be kept in the downstream TiDB, then additional configuration `filter-rules` can be used to filter out these operations.

For such scenarios, please refer to [Data Migration Simple Use Scenario](usage-scenario-simple-replication.md).

## Shard-merge scenario

If there are multiple sharding tables in the upstream MySQL/MariaDB, it is usually expected to merge them into only one table when migrating to TiDB. The schema name or the table name in the upstream data can be renamed by configuring `route-rules`, and then these tables will be merged into the same downstream schema or table. For details, please refer to [Data Migration Shard Merge Scenario](usage-scenario-shard-merge.md) and [Best Practices of Data Migration in the Shard Merge Scenario](shard-merge-best-practices.md).

DM supports the migration of DDL specifically. For details, please refer to [Merge shard table](feature-shard-merge.md).

If you only need to migrate part of the business data or filter some specific operations, you can directly refer to [Migrating some business data in MySQL/MariaDB](#migrating-some-business-data-in-mysql/mariadb) in part `Non-merging sharding table scenario`.

## Online DDL scenario

In MySQL's ecosystem, tools such as pt-osc or gh-ost are often used to perform online DDL operations. DM provides special support for such scenarios, which can be enabled by configuring the `online-ddl-scheme` parameter. For details, please refer to [DM online-ddl-scheme](online-ddl-scheme.md).

## Change upstream MySQL instance of DM connection

When using MySQL/MariaDB, a primary-secondary cluster is often built to improve read performance and ensure data security. If the upstream MySQL/MariaDB instance connected by DM-worker is unavailable for some reason when using DM to migration data, you need to connect DM-worker to another instance in the upstream primary-secondary cluster. For such scenarios, DM provides nice support, but there are still some limitations. For details, please refer to [Switching the connection between DM-worker and the upstream MySQL instance] (usage-scenario-master-slave-switch.md).