# TiDB Data Migration (DM) Documentation

<!-- markdownlint-disable MD007 -->
<!-- markdownlint-disable MD032 -->

## TOC

+ About DM
  + Benchmarks
    - [DM 1.0-GA Benchmark Report](benchmark-v1.0-ga.md)
    - [DM 1.0-alpha Benchmark Report](benchmark-v1-alpha.md)
  + Features
    - [Table Routing](key-features.md#table-routing)
    - [Block and Allow Lists](key-features.md#block-and-allow-table-lists)
    - [Binlog Event Filter](key-features.md#binlog-event-filter)
    - [Replication Delay Monitoring](key-features.md#replication-delay-monitoring)
    - [Online DDL Scheme](feature-online-ddl-scheme.md)
    - [Merge and Replicate Data from Sharded Tables](feature-shard-merge.md)
+ [Usage Scenarios](scenarios.md)
+ Quick Start
  - [Deploy a DM cluster](quick-start-with-dm.md)
  - [Create Data Migration Task](quick-start-create-task.md)
+ Deploy
  + [Software and Hardware Requirements](hardware-and-software-requirements.md)
  + Deploy a DM Cluster
    - [Use DM-Ansible](deploy-a-dm-cluster-using-ansible.md)
    - [Use Binary](deploy-a-dm-cluster-using-binary.md)
  - [Replicate Data Using DM](replicate-data-using-dm.md)
  - [Monitor](monitor-a-dm-cluster.md)
  - [Performance Test](performance-test.md)
+ Maintain
  + Cluster Upgrade
    - [Manually Upgrade from v1.0.x to v2.0.x](manually-upgrade-dm-1.0-to-2.0.md)
    - [Upgrade Between v1.0.x](upgrade-dm-1.0.md)
  - [Cluster Operations](cluster-operations.md)
  + Manage Replication Tasks
    - [dmctl Introduction](dmctl-introduction.md)
    - [Manage Upstream Data Source](manage-source.md)
    - [Precheck a Task](precheck.md)
    - [Create a Task](create-task.md)
    - [Query Status](query-status.md)
    - [Query Error](query-error.md)
    - [Pause a Task](pause-task.md)
    - [Resume a Task](resume-task.md)
    - [Stop a Task](stop-task.md)
    - [Skip or Replace Abnormal SQL Statements](skip-or-replace-abnormal-sql-statements.md)
  - [Manually Handle Sharding DDL Locks](feature-manually-handling-sharding-ddl-locks.md)
  - [Handle Alerts](handle-alerts.md)
  - [Daily Check](daily-check.md)
+ Troubleshoot
  - [Handle Errors](error-handling.md)
  - [Handle Performance Issues](handle-performance-issues.md)
+ Tutorials
  - [Simple Replication Scenario](usage-scenario-simple-replication.md)
  - [Shard Merge Scenario](usage-scenario-shard-merge.md)
  - [Migrate from Amazon Aurora](migrate-from-mysql-aurora.md)
  - [Shard Merge Best Practices](shard-merge-best-practices.md)
  - [Switch DM-worker Connection between MySQL Instances](usage-scenario-master-slave-switch.md)
+ Performance Tuning
  - [Optimize Configuration](tune-configuration.md)
+ Reference
  + Architecture
    - [DM Overview](overview.md)
    - [DM-worker](dm-worker-intro.md)
    - [DM Relay Log](relay-log.md)
  - [Command-line Flags](command-line-flags.md)
  + Configuration
    - [Overview](config-overview.md)
    - [DM-master Configuration](dm-master-configuration-file.md)
    - [DM-worker Configuration](dm-worker-configuration-file.md)
    - [Upstream Database Configuration](source-configuration-file.md)
    - [Task Configuration](task-configuration-file.md)
    - [Full Task Configuration](task-configuration-file-full.md)
    - [Use DM Portal](dm-portal.md)
  - [Monitoring Metrics](monitor-a-dm-cluster.md)
  - [Alert Rules](alert-rules.md)
  - [Error Codes](error-handling.md#handle-common-errors)
+ [FAQ](faq.md)
+ [Glossary](glossary.md)
+ Release Notes
  + v1.0
    - [1.0.6](releases/1.0.6.md)
    - [1.0.5](releases/1.0.5.md)
    - [1.0.4](releases/1.0.4.md)
    - [1.0.3](releases/1.0.3.md)
    - [1.0.2](releases/1.0.2.md)
