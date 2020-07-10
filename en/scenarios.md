---
title: Usage Scenarios
summary: Understand the main usage scenarios of TiDB Data Migration.
category: reference
---

# Usage Scenarios

This document introduces the main usage scenarios of TiDB Data Migration (DM) and the usage suggestions.

## Non-shard-merge scenario

### Use TiDB as MySQL/MariaDB's secondary database

If you want to use TiDB as the upstream MySQL/MariaDB's secondary library, that is, import the full data of the upstream instance to TiDB, and replicate the incremental data to TiDB in real-time once data changes are made, you can configure the data migration task according to the following rules:

- Specify `task-mode` as `all`.
- Configure `target-database` based on the connection information of the downstream TiDB.
- Configure `source-id` in `mysql-instances` based on the ID information of the upstream MySQL/MariaDB instances.
- Use the default setting of concurrency control parameters or configure `mydumper-thread`, `loader-thread`, and `syncer-thread` as needed.
- No need to configure `route-rules`, `filter-rules` and `black-white-list`.

### Migrate some application data from MySQL/MariaDB

If there are multiple application data in MySQL/MariaDB, but only part of the data needs to be migrated to TiDB, you can configure the migration task by referring to [Use TiDB as a secondary library of MySQL/MariaDB](#use-tidb-as-mysqlmariadbs-secondary-database). Then configure `black-white-list` as needed.

To migrate upstream data to a schema or table with a different name in the downstream database, you can configure `route-rules`.

For some archiving scenarios, some data may be periodically cleaned up by executing the `TRUNCATE TABLE`/`DROP TABLE` command  in the upstream or through other means. To make sure all data are retained in the downstream TiDB, you can disable these data clearing operations by configuring `filter-rules`.

For more information, refer to [Data Migration Simple Usage Scenario](usage-scenario-simple-replication.md).

## Shard Merge scenario

If there are multiple sharded tables in multiple sharded schemas in the upstream MySQL/MariaDB, to merge them into one table or schema when migrating to TiDB, you can rename the table name or the schema name in the upstream database by configuring `route-rules`, and then these tables can be merged into the same downstream schema or table. For details, refer to [Data Migration Shard Merge Scenario](usage-scenario-shard-merge.md) and [Best Practices of Data Migration in the Shard Merge Scenario](shard-merge-best-practices.md).

Specifically, DM supports the migration of DDL. For details, refer to [Merge and Replicate Data from Sharded Tables](feature-shard-merge.md).

If you only need to migrate some application data or filter out some operations, refer to [Migrate some applicationb data from MySQL/MariaDB](#migrate-some-application-data-from-mysqlmariadb) in part `Non-merging sharding table scenario`.

## Online DDL scenario

In MySQL ecosystem, tools such as pt-osc or gh-ost are often used to perform online DDL operations. DM also supports such scenarios. You can enable pt-osc and gh-ost by configuring the `online-ddl-scheme` parameter. For details, refer to [DM Online DDL Scheme](online-ddl-scheme.md).

## Switch the DM-worker connection to another instance

When you use MySQL/MariaDB, a primary-secondary cluster can be built to improve the read performance and to ensure data safety. If the upstream MySQL/MariaDB instance connected by DM-worker is unavailable for some reason when you use DM to migrate data, you need to switch the DM-worker connection to another MySQL/MariaDB instance in the upstream primary-secondary cluster. For such scenarios, DM provides good support, but there are some limitations.
