---
title: Alert Handling
summary: Understand the handling methods of alert information in DM.
category: reference
---

# Alert Handling

This document introduces how to deal with the alert information in DM.

## Alert rules related to task status

### `DM_task_state`

When the sub-task of DM-worker is in the `Paused` status for over 20 minutes, an alert is triggered. You need to refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to relay log

### `DM_relay_process_exits_with_error`

This alert triggered and the relay log processing unit's status becomes to `Paused` when the relay log processing unit has some error. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_remain_storage_of_relay_log`

This alert triggered when the disk free space of relay log was less than 10G, and its handling method below.

- Delete unwanted data by manual to increase disk free space.
- Try to adjust the [automatic data purge policy of relay log](relay-log.md#Automatic data purge) or execute [manual data purge](relay-log.md#Manual-data-purge).
- Try to [migrate DM-worker instance](cluster-operations.md#Replace/migrate-a-DM-master-instance) to another disk which has enough free space.

### `DM_relay_log_data_corruption`

This alert triggered immediately and the relay log processing unit's status becomes to `Paused` when checksum can not pass while the relay log processing unit processing binlog event which comes from upstream. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_fail_to_read_binlog_from_master`

This alert triggered immediately and the relay log processing unit's status becomes to `Paused` when some error occurs while the relay log processing unit tries to read binlog event from upstream. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_fail_to_write_relay_log`

This alert triggered immediately and the relay log processing unit's status becomes to `Paused` when some error occurs while the relay log processing unit tries to write binlog event into the relay log file. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_binlog_file_gap_between_master_relay`

This alert triggered when the number of latest binlog files extracted by the relay log processing unit lags behind the current upstream MySQL/MariaDB number by 1 (excluding 1) for over 10 minutes. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

## The alert of Dump/Load

### `DM_dump_process_exists_with_error`

This alert triggered and the dump processing unit'status becomes to `Paused` when the dump processing unit has some error. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_load_process_exists_with_error`

This alert triggered and the load processing unit'status becomes to `Paused` when the load processing unit has some error. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

## The alert of binlog replication

### `DM_sync_process_exists_with_error`

This alert triggered immediately and the binlog replication processing unit'status becomes to `Paused` when the binlog replication processing unit has some error. You need to refer to [DM error handling](error-handling.md#Data-Migration-Error-Handling).

### `DM_binlog_file_gap_between_master_syncer`

This alert triggered when the number of latest binlog files processed by the binlog replication processing unit lags behind the current upstream MySQL/MariaDB number by 1 (excluding 1) for over 10 minutes. You need to refer to [handle performance issues](handle-performance-issues.md).

### `DM_binlog_file_gap_between_relay_syncer`

This alert triggered when the number of latest binlog files processed by the binlog replication processing unit lags behind the relay log processing unit number by 1 (excluding 1) for over 10 minutes. You need to refer to [handle performance issues](handle-performance-issues.md).
