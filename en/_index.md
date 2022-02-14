---
title: TiDB Data Migration Documentation
summary: Learn about TiDB Data Migration documentation.
---

# TiDB Data Migration Documentation

[TiDB Data Migration](https://github.com/pingcap/tiflow/tree/master/dm) (DM) is a convenient data migration tool, which supports data migration from MySQL-compatible databases (such as MySQL, MariaDB, and Aurora MySQL) into TiDB. It can help to reduce the operation cost of data migration.

> **Note:**
>
> - Since DM v5.4, the TiDB Data Migration documentation has been merged into the TiDB documentation of the same version. To read the DM documentation of v5.4 or a later version, go to the [TiDB Documentation](https://docs.pingcap.com/zh/tidb/stable/dm-overview) of the same version.
> - Since October 2021, DM's GitHub repository has been moved to [pingcap/tiflow](https://github.com/pingcap/tiflow/tree/master/dm). If you see any issues with DM, submit your issue to the `pingcap/tiflow` repository for feedback.
> - In earlier versions (v1.0 and v2.0), DM uses version numbers that are independent of TiDB. Since v5.3, DM uses the same version number as TiDB. The next version of DM v2.0 is DM v5.3. There are no compatibility changes from DM v2.0 to v5.3, and the upgrade process is no different from a normal upgrade, only an increase in version number.

<NavColumns>
<NavColumn>
<ColumnTitle>About TiDB Data Migration</ColumnTitle>

- [What is DM?](dm-overview.md)
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

- [Software and Hardware Requirements](dm-hardware-and-software-requirements.md)
- [Deploy DM Using TiUP (Recommended)](deploy-a-dm-cluster-using-tiup.md)
- [Deploy DM Using TiUP Offline](deploy-a-dm-cluster-using-tiup-offline.md)
- [Deploy DM Using Binary](deploy-a-dm-cluster-using-binary.md)
- [Use DM to Migrate Data](migrate-data-using-dm.md)
- [DM Performance Test](dm-performance-test.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Maintain</ColumnTitle>

- [Maintain DM Clusters Using TiUP (Recommended)](maintain-dm-using-tiup.md)
- [Maintain DM Clusters Using dmctl](dmctl-introduction.md)
- [Upgrade DM](manually-upgrade-dm-1.0-to-2.0.md)
- [Manually Handle Sharding DDL Locks](manually-handling-sharding-ddl-locks.md)
- [Handle Alerts](dm-handle-alerts.md)
- [Daily Check](dm-daily-check.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Usage Scenarios</ColumnTitle>

- [Migrate from Aurora](migrate-from-mysql-aurora.md)
- [Switch the MySQL Instance to Be Migrated](usage-scenario-master-slave-switch.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Reference</ColumnTitle>

- [DM Architecture](dm-arch.md)
- [Configuration File Overview](dm-config-overview.md)
- [Monitoring Metrics and Alerts](monitor-a-dm-cluster.md)
- [Error Handling](dm-error-handling.md)

</NavColumn>

</NavColumns>
