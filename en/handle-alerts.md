---
title: Alert Handling
summary: Understand the handling methods of alert information in DM.
---

# Alert Handling

This document introduces how to deal with the alert information in DM.

## Alert rules related to task status

### `DM_task_state`

When the sub-task of DM-worker is in the `Paused` status for over 20 minutes, an alert is triggered. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to relay log

### `DM_relay_process_exits_with_error`

When the relay log processing unit encounters an error, this unit moves to `Paused` state, and an alert is triggered immediately. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_remain_storage_of_relay_log`

When the free space of the disk where the relay log is located is less than 10G, an alert is triggered. You can take the following methods to handle the alert:

- Delete unwanted data manually to increase free disk space.
- Reconfigure the [automatic data purge strategy of the relay log](relay-log.md#automatic-data-purge) or[purge data manually](relay-log.md#manual-data-purge).
- [Migrate the DM-worker instance](cluster-operations.md#replacemigrate-a-dm-worker-instance) to a disk with enough free space.

### `DM_relay_log_data_corruption`

When checksum can not pass while the relay log processing unit processing binlog event which comes from upstream, an alert is triggered immediately and the relay log processing unit's status becomes to `Paused`. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_fail_to_read_binlog_from_master`

If an error occurs when the relay log processing unit tries to read binlog event from the upstream, an alert is triggered immediately and this unit moves to the `Paused` status. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_fail_to_write_relay_log`

If an error occurs when the relay log processing unit tries to write binlog event into the relay log file, an alert is triggered immediately and this unit moves to the `Paused` status. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_binlog_file_gap_between_master_relay`

When the number of latest binlog files extracted by the relay log processing unit lags behind the current upstream MySQL/MariaDB number by 1 (excluding 1) for over 10 minutes, an alert is triggered. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

## Alert rules related to Dump/Load

### `DM_dump_process_exists_with_error`

When the Dump processing unit encounters an error, an alert is triggered and this unit moves to the `Paused` status. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_load_process_exists_with_error`

When the Load processing unit encounters an error, an alert is triggered and this unit moves to the `Paused` state. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to binlog replication

### `DM_sync_process_exists_with_error`

When the binlog replication processing unit has some error, an alert is triggered immediately and the binlog replication processing unit'status becomes to `Paused` . You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_binlog_file_gap_between_master_syncer`

When the number of latest binlog files processed by the binlog replication processing unit lags behind the current upstream MySQL/MariaDB number by 1 (excluding 1) for over 10 minutes, an alert is triggered. You need to refer to [handle performance issues](handle-performance-issues.md).

### `DM_binlog_file_gap_between_relay_syncer`

When the number of latest binlog files processed by the binlog replication processing unit lags behind the relay log processing unit number by 1 (excluding 1) for over 10 minutes, an alert is triggered. You need to refer to [handle performance issues](handle-performance-issues.md).
