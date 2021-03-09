---
title: Handle Alerts
summary: Understand how to deal with the alert information in DM.
---

# Handle Alerts

This document introduces how to deal with the alert information in DM.

## Alert rules related to task status

### `DM_task_state`

- Description:

    When a sub-task of DM-worker is in the `Paused` state for over 20 minutes, an alert is triggered.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to relay log

### `DM_relay_process_exits_with_error`

- Description:

    When the relay log processing unit encounters an error, this unit moves to `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_remain_storage_of_relay_log`

- Description:

    When the free space of the disk where the relay log is located is less than 10G, an alert is triggered.

- Solutions:

    You can take the following methods to handle the alert:

    - Delete unwanted data manually to increase free disk space.
    - Reconfigure the [automatic data purge strategy of the relay log](relay-log.md#automatic-data-purge) or [purge data manually](relay-log.md#manual-data-purge).
    - [Migrate the DM-worker instance](cluster-operations.md#replacemigrate-a-dm-worker-instance) to a disk with enough free space.

### `DM_relay_log_data_corruption`

- Description:

    When the relay log processing unit validates the binlog event read from the upstream and detects abnormal checksum information, this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_fail_to_read_binlog_from_master`

- Description:

    If an error occurs when the relay log processing unit tries to read the binlog event from the upstream, this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_fail_to_write_relay_log`

- Description:

    If an error occurs when the relay log processing unit tries to write the binlog event into the relay log file, this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_binlog_file_gap_between_master_relay`

- Description:

    When the number of the binlog files in the current upstream MySQL/MariaDB exceeds that of the latest binlog files pulled by the relay log processing unit by **more than** 1 for 10 minutes, and an alert is triggered.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to Dump/Load

### `DM_dump_process_exists_with_error`

- Description:

    When the Dump processing unit encounters an error, this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_load_process_exists_with_error`

- Description:

    When the Load processing unit encounters an error, this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

## Alert rules related to binlog replication

### `DM_sync_process_exists_with_error`

- Description:

    When the binlog replication processing unit encounters an error,  this unit moves to the `Paused` state, and an alert is triggered immediately.

- Solution:

    Refer to [Troubleshoot DM](error-handling.md#troubleshooting).

### `DM_binlog_file_gap_between_master_syncer`

- Description:

    When the number of the binlog files in the current upstream MySQL/MariaDB exceeds that of the latest binlog files processed by the relay log processing unit by **more than** 1 for 10 minutes, an alert is triggered.

- Solution:

    Refer to [Handle Performance Issues](handle-performance-issues.md).

### `DM_binlog_file_gap_between_relay_syncer`

- Description:

    When the number of the binlog files in the current relay log processing unit exceeds that of the latest binlog files processed by the binlog replication processing unit by **more than** 1 for 10 minutes, an alert is triggered.

- Solution:

    Refer to [Handle Performance Issues](handle-performance-issues.md).
