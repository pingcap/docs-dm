# TiDB Data Migration (DM) Documentation

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## TOC

+ About DM
  + Benchmarks
    - [DM 2.0-GA Benchmark Report](benchmark-v2.0-ga.md)
  + Features
    - [Table Routing](key-features.md#table-routing)
    - [Block and Allow Lists](key-features.md#block-and-allow-table-lists)
    - [Binlog Event Filter](key-features.md#binlog-event-filter)
    - [Online DDL Scheme](feature-online-ddl-scheme.md)
    + Merge and Migrate Data from Sharded Tables
      - [Overview](feature-shard-merge.md)
      - [Pessimistic Mode](feature-shard-merge-pessimistic.md)
      - [Optimistic Mode](feature-shard-merge-optimistic.md)
+ [Usage Scenarios](scenarios.md)
+ Quick Start
  - [Deploy a DM Cluster](quick-start-with-dm.md)
  - [Create Data Migration Task](quick-start-create-task.md)
+ Deploy
  + [Software and Hardware Requirements](hardware-and-software-requirements.md)
  + Deploy a DM Cluster
    - [Use TiUP](deploy-a-dm-cluster-using-tiup.md)
    - [Use TiUP Offline](deploy-a-dm-cluster-using-tiup-offline.md)
    - [Use Binary](deploy-a-dm-cluster-using-binary.md)
  - [Migrate Data Using DM](migrate-data-using-dm.md)
  - [Monitor](monitor-a-dm-cluster.md)
  - [Performance Test](performance-test.md)
+ Maintain
  + Cluster Upgrade
    - [Manually Upgrade from v1.0.x to v2.0.x](manually-upgrade-dm-1.0-to-2.0.md)
    - [Upgrade Between v1.0.x](upgrade-dm-1.0.md)
  - [Use TiUP to Maintain a DM Cluster](maintain-dm-using-tiup.md)
  + Manage Migration Tasks
    - [dmctl Introduction](dmctl-introduction.md)
    - [Manage Upstream Data Source](manage-source.md)
    - [Precheck a Task](precheck.md)
    - [Create a Task](create-task.md)
    - [Query Status](query-status.md)
    - [Pause a Task](pause-task.md)
    - [Resume a Task](resume-task.md)
    - [Stop a Task](stop-task.md)
    - [Handle Failed SQL Statements](handle-failed-sql-statements.md)
  - [Manually Handle Sharding DDL Locks](manually-handling-sharding-ddl-locks.md)
  - [Manage Table Schema during Migration](manage-schema.md)
  - [Handle Alerts](handle-alerts.md)
  - [Daily Check](daily-check.md)
+ Troubleshoot
  - [Handle Errors](error-handling.md)
  - [Handle Performance Issues](handle-performance-issues.md)
+ Tutorials
  - [Data Migration Simple Usage Scenario](usage-scenario-simple-migration.md)
  - [Shard Merge Scenario](usage-scenario-shard-merge.md)
  - [Migrate from a MySQL-compatible Database](migrate-from-mysql-aurora.md)
  - [Shard Merge Best Practices](shard-merge-best-practices.md)
  - [Migration When the Downstream Table Has More Columns](usage-scenario-downstream-more-columns.md)
  - [Switch DM-worker Connection between MySQL Instances](usage-scenario-master-slave-switch.md)
+ Performance Tuning
  - [Optimize Configuration](tune-configuration.md)
+ Reference
  + Architecture
    - [DM Overview](overview.md)
    - [DM-worker](dm-worker-intro.md)
  - [Command-line Flags](command-line-flags.md)
  + Configuration
    - [Overview](config-overview.md)
    - [DM-master Configuration](dm-master-configuration-file.md)
    - [DM-worker Configuration](dm-worker-configuration-file.md)
    - [Upstream Database Configuration](source-configuration-file.md)
    - [Task Configuration](task-configuration-file.md)
    - [Full Task Configuration](task-configuration-file-full.md)
  + Secure
    - [Enable TLS for DM Connections](enable-tls.md)
    - [Generate Self-signed Certificates](generate-self-signed-certificates.md)
  - [Monitoring Metrics](monitor-a-dm-cluster.md)
  - [Alert Rules](alert-rules.md)
  - [Error Codes](error-handling.md#handle-common-errors)
+ [FAQ](faq.md)
+ [Glossary](glossary.md)
+ Release Notes
  + v2.0
    - [2.0 GA](releases/2.0.0-ga.md)
    - [2.0.0-rc.2](releases/2.0.0-rc.2.md)
    - [2.0.0-rc](releases/2.0.0-rc.md)
  + v1.0
    - [1.0.6](releases/1.0.6.md)
    - [1.0.5](releases/1.0.5.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.2](releases/1.0.2.md)
