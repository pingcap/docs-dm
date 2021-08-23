---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
---

# TiDB Data Migration Documentation   

[TiDB Data Migration](https://github.com/pingcap/dm) (DM) is an integrated data migration task management platform, which supports data migration from MySQL-compatible databases (such as MySQL, MariaDB, and Aurora MySQL) into TiDB. It can help to reduce the operation cost of data migration.

The latest stable version of DM is v2.0.

**What's new in DM v2.0:**

- [High availability of data migration tasks](dm-arch.md#high-availability). The data migration task can run normally even when some DM-master or DM-worker nodes fail.
- [Sharding DDL support in the optimistic mode](feature-shard-merge-optimistic.md). In this mode, migration latency can be reduced in some scenarios and you can make A/B changes in the upstream database.
- Better usability, including the new [error handling mechanism](handle-failed-ddl-statements.md) and the easier-to-read error messages and error handling suggestions.
- [TLS support](enable-tls.md) for connections between the upstream and the downstream, and for connections between DM components.
- Support for migrating data from MySQL 8.0 (experimental).

Currently, DM does not have a graphical interface and provides limited support for managing a large number of data migration tasks. **If you need to operate a large number of data migration tasks, using DM might be inconvenient.**

<NavColumns>
<NavColumn>
<ColumnTitle>About TiDB Data Migration</ColumnTitle>

- [What is DM?](overview.md)
- [DM Architecture](dm-arch.md)
- [Performance](benchmark-v2.0-ga.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Quick Start</ColumnTitle>

- [Quick Start](quick-start-with-dm.md)
- [Deploy a DM cluster Using TiUP](deploy-a-dm-cluster-using-tiup.md)
- [Create a Data Source](quick-start-create-source.md)
- [Create a Data Migration Task](quick-create-migration-task.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Deploy and Use</ColumnTitle>

- [Software and Hardware Requirements](hardware-and-software-requirements.md)
- [Deploy DM Using TiUP (Recommended)](deploy-a-dm-cluster-using-tiup.md)
- [Deploy DM Using TiUP Offline](deploy-a-dm-cluster-using-tiup-offline.md)
- [Deploy DM Using Binary](deploy-a-dm-cluster-using-binary.md)
- [Use DM to Migrate Data](migrate-data-using-dm.md)
- [DM Performance Test](performance-test.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Maintain</ColumnTitle>

- [Maintain DM Clusters Using TiUP (Recommended)](maintain-dm-using-tiup.md)
- [Maintain DM Clusters Using dmctl](dmctl-introduction.md)
- [Upgrade DM](manually-upgrade-dm-1.0-to-2.0.md)
- [Manually Handle Sharding DDL Locks](manually-handling-sharding-ddl-locks.md)
- [Handle Alerts](handle-alerts.md)
- [Daily Check](daily-check.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Usage Scenarios</ColumnTitle>

- [Migrate from Aurora](migrate-from-mysql-aurora.md)
- [Switch the MySQL Instance to Be Migrated](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Reference</ColumnTitle>

- [DM Architecture](dm-arch.md)
- [Configuration File Overview](config-overview.md)
- [Monitoring Metrics and Alerts](monitor-a-dm-cluster.md)
- [Error Handling](error-handling.md)

</NavColumn>

</NavColumns>
