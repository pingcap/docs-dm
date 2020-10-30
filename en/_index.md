---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
aliases: ['/docs/tidb-data-migration/dev/']
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform that supports the full data migration and the incremental data replication from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

**What's new in DM v2.0:**

- [High availability of data migration tasks](overview.md#high-availability). The data migration task can run normally even when some DM-master or DM-worker nodes fail.
- [Sharding DDL support in the optimistic mode](feature-shard-merge-optimistic.md). In this mode, migration latency can be reduced in some scenarios and you can make A/B changes in the upstream database.
- Better usability, including the new [error handling mechanism](handle-failed-sql-statements.md) and the easier-to-read error messages and error handling suggestions.
- [TLS support](enable-tls.md) for connections between the upstream and the downstream, and for connections between DM components.
- Support for migrating data from MySQL 8.0 (experimental).

<NavColumns>
<NavColumn>
<ColumnTitle>About TiDB Data Migration</ColumnTitle>

- [Performance](benchmark-v2.0-ga.md)
- [Table routing](key-features.md#table-routing)
- [Block & Allow Lists](key-features.md#block-and-allow-table-lists)
- [Binlog Event Filter](key-features.md#binlog-event-filter)
- [Online DDL Scheme](feature-online-ddl-scheme.md)
- [Shard Merge](feature-shard-merge.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Quick Start</ColumnTitle>

- [Usage Scenarios](usage-scenario-shard-merge.md)
- [Quick Start with DM](quick-start-with-dm.md)
- [Migrate Data Using DM](migrate-data-using-dm.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Deploy and Use</ColumnTitle>

- [Deploy DM Using TiUP](deploy-a-dm-cluster-using-tiup.md)
- [Deploy DM Using TiUP Offline](deploy-a-dm-cluster-using-tiup-offline.md)
- [Deploy DM Using Binary](deploy-a-dm-cluster-using-binary.md)
- [Use DM to Migrate Data](migrate-data-using-dm.md)
- [Monitor and Alert](monitor-a-dm-cluster.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Maintain</ColumnTitle>

- [Upgrade DM](manually-upgrade-dm-1.0-to-2.0.md)
- [DM Cluster Operations](maintain-dm-using-tiup.md)
- [Create a Task](create-task.md)
- [Pause a Task](pause-task.md)
- [Resume a Task](resume-task.md)
- [Stop a Task](stop-task.md)
- [Manually Handle Sharding DDL Locks](manually-handling-sharding-ddl-locks.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Tutorials</ColumnTitle>

- [Data Migration Simple Usage Scenario](usage-scenario-simple-migration.md)
- [Shard Merge Scenario](usage-scenario-shard-merge.md)
- [Migrate from a MySQL-compatible Database](migrate-from-mysql-aurora.md)
- [Shard Merge Scenario Best Practices](shard-merge-best-practices.md)
- [Switch DM-worker connection between MySQL Instances](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Reference</ColumnTitle>

- [DM Architecture](overview.md)
- [Configuration File Overview](config-overview.md)
- [Monitoring Metrics and Alerts](monitor-a-dm-cluster.md)
- [Error Handling](error-handling.md)

</NavColumn>

</NavColumns>
