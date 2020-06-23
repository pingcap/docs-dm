---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data replication task management platform that supports the full data migration and the incremental data migration from MySQL/MariaDB into TiDB. It can help to reduce the operations cost and simplify the troubleshooting process.

<NavColumns>
<NavColumn>
<ColumnTitle>About TiDB Data Migration</ColumnTitle>

- [Performance](benchmark-v1.0-ga.md)
- [Table routing](feature-overview.md#table-routing)
- [Black & White Lists](feature-overview.md#black-and-white-table-lists)
- [Binlog Event Filter](feature-overview.md#binlog-event-filter)
- [Replication delay monitoring](feature-overview.md#replication-delay-monitoring)
- [Online DDL Scheme](online-ddl-scheme.md)
- [Shard Merge](feature-shard-merge.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Quick Start</ColumnTitle>

- [Usage Scenarios](usage-scenario-shard-merge.md)
- [Quick Start with DM](quick-start-with-dm.md)
- [Replicate Data Using DM](replicate-data-using-dm.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Deploy and Use</ColumnTitle>

- [Deploy DM Using DM-Ansible](deploy-a-dm-cluster-using-ansible.md)
- [Deploy DM Using Binary](deploy-a-dm-cluster-using-binary.md)
- [Use DM to Replicate Data](replicate-data-using-dm.md)
- [Monitor and Alert](monitor-a-dm-cluster.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Maintain</ColumnTitle>

- [Upgrade DM](dm-upgrade.md)
- [DM Cluster Operations](cluster-operations.md)
- [Manage Replication Tasks](manage-replication-tasks.md)
- [Manually Handle Sharding DDL Locks](feature-manually-handling-sharding-ddl-locks.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Tutorials</ColumnTitle>

- [Simple Usage Scenario](usage-scenario-simple-replication.md)
- [Shard Merge Scenario](usage-scenario-shard-merge.md)
- [Migrate from MySQL (Amazon Aurora)](migrate-from-mysql-aurora.md)
- [Shard Merge Scenario Best Practices](shard-merge-best-practices.md)
- [Switch DM-worker connection](cluster-operations.md#switch-dm-worker-connection-between-upstream-mysql-instances)

</NavColumn>

<NavColumn>
<ColumnTitle>Reference</ColumnTitle>

- [DM Architecture](overview.md)
- [Configuration File Overview](config-overview.md)
- [Monitoring Metrics and Alerts](monitor-a-dm-cluster.md)
- [Error Handling](error-handling.md)

</NavColumn>

</NavColumns>
