---
title: Create a data migration task
summary: Learn how to configure a data migration task in different scenarios.
---

# Create a data migration task

> **Note:**
>
> Before creating a data migration task, you need to perform the following operations:
>
> 1. [Deploy a DM Cluster Using TiUP](deploy-a-dm-cluster-using-tiup.md)。
> 2. [Create a Data Source](quick-start-create-source.md)。

This document introduces how to configure a data migration task in different scenarios. You can choose suitable documents to create your data migration task according to the specific scenario.

In addition to documents based on different scenarios, you can also refer to the following documents:

- For a complete example of data migration task configuration, refer to [DM Advanced Task Configuration File](task-configuration-file-full.md).
- For a data migration task configuration guide, refer to [Data Migration Task Configuration Guide](task-configuration-guide.md).

## Migrate Data from Multiple Data Sources to TiDB

If you need to migrate data from multiple data sources to TiDB, or to avoid some DDL/DML operations of some tables, refer to 如果你需要将多个数据源的数据汇总迁移到 TiDB，此外还需要进行表重命名以防止多个数据源中相同表名在迁移过程中出现冲突，[Migrate Data from Multiple Data Sources to TiDB](usage-scenario-simple-migration.md).

## Migrate Sharded Schemas and Sharded Tables to TiDB

If you need to migrate sharded schemas and sharded tables to TiDB, refer to [Data Migration Shard Merge Scenario](usage-scenario-shard-merge.md).

## Migrate Incremental Data to TiDB

If you have already migrated full data migration using other tools like TiDB Lightning and you need to migrate incremental data, refer to [Migrate Incremental Data to TiDB](usage-scenario-incremental-migration.md).

## Migration when the Downstream Table Has More Columns

If you need to customize your table schema and TiDB table schema has more columns than the data source table, refer to [Migration when the Downstream Table Has More Columns](usage-scenario-downstream-more-columns.md).
