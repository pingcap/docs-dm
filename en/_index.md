---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
aliases: ['/docs/tidb-data-migration/dev/']
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/ticdc/tree/master/dm) (DM) is a convenient data migration tool, which supports data migration from MySQL-compatible databases (such as MySQL, MariaDB, and Aurora MySQL) into TiDB. It can help to reduce the operation cost of data migration.

The latest stable version of DM is v5.3.

**What's new in DM 5.3:**

- Compact multiple updates on a single row into one statement and merge batch updates of multiple rows into one statement to make sure low-latency replication from MySQL to TiDB.
- Add DM OpenAPI to better maintain DM clusters (experimental).

**Notes**

- DM's github repository has been moved to [https://github.com/pingcap/ticdc/tree/master/dm](https://github.com/pingcap/ticdc/tree/master/dm). You can go to the new repository to submit issue for follow-up feedback.
- The DM release schedule will be aligned with TiCDC, so the version number is also keep same on both sides, the planned v2.1.0 release will be changed to v5.3.0.

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
