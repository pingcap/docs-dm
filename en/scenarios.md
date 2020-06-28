---
title: Application scenario
summary: Understand the main application scenarios supported by TiDB Data Migration.
category: reference
---

# Application scenarios

This document introduces the main application scenarios supported by TiDB Data Migration (DM) and related usage suggestions.

## Non-combined library combined table scene

### TiDB as MySQL/MariaDB secondary library

If you want to use TiDB as the upstream MySQL/MariaDB secondary library, that is, import all the data in the upstream instance to TiDB in full form, and then copy the subsequent changes to TiDB in incremental form in real time, simply configure the data migration task according to the following rules: can:

-Specify `task-mode` as `all`.
-Configure `target-database` as downstream TiDB related connection information.
-Configure the `source-id` for upstream MySQL/MariaDB in `mysql-instances`.
-Use default concurrency control parameters or configure `mydumper-thread`, `loader-thread` and `syncer-thread` as needed.
-No need to configure `route-rules`, `filter-rules` and `block-allow-list` etc.

### Migrating some business data in MySQL/MariaDB

If there are multiple business data in the original MySQL/MariaDB, but only part of the business data needs to be migrated to TiDB for the time being, then follow [Use TiDB as a secondary library of MySQL/MariaDB] (#将-tidb-为-mysqlmariadb-的From the library) After configuring the migration task, configure `block-allow-list` as needed.

If you want to migrate upstream data to a library or table with a different name downstream, you can additionally configure `route-rules`.

For some archiving scenarios, some data may be periodically cleaned up through `TRUNCATE TABLE`/`DROP TABLE` or other methods in the upstream, but it is expected that all data will be kept in the downstream TiDB, then additional configuration `filter-rules` can be used to filter out Related data cleaning operations.

For such scenarios, please refer to [Data Migration Simple Use Scenario] (usage-scenario-simple-replication.md).

## 合库合表 scene

If the data in the upstream MySQL/MariaDB exists in the form of sub-library and table, it is usually expected to merge it when migrating to TiDB. At this time, the library name in the upstream data can be configured by configuring `route-rules` , Table names, etc. are renamed and merged into the same downstream library or table. For details, please refer to [DM Sub-database Sub-table Merging Scenario] (usage-scenario-shard-merge.md) and [Sub-table Merging Data Migration Best Practices ](shard-merge-best-practices.md).

For the migration of DDL, DM provides special support. For details, please refer to [Sub-database sub-table merge synchronization] (feature-shard-merge.md).

If you only need to migrate part of the business data or filter some specific operations, you can directly refer to [Migrate some business data in MySQL/MariaDB] in the aforementioned non-synthetic database merge table scenario (#migrated-mysqlmariadb-partial business data).

## online DDL scene

In the MySQL ecosystem, tools such as pt-osc or gh-ost are often used to perform online DDL operations. DM provides special support for such scenarios, which can be enabled by configuring the `online-ddl-scheme` parameter. For details, please refer to [DM online-ddl-scheme](feature-online-ddl-scheme.md).

## Change upstream MySQL instance of DM connection

When using MySQL/MariaDB, a primary-secondary cluster is often built to improve read performance and ensure data security. If during the data migration process using DM, the upstream MySQL/MariaDB instance originally connected by DM-worker is unavailable for some reason, you need to connect DM-worker to another instance between the upstream primary and secondary clusters. For such scenarios, DM provides better support, but there are still some limitations. For details, please refer to [Switching the connection between DM-worker and the upstream MySQL instance] (usage-scenario-primary-secondary-switch.md).