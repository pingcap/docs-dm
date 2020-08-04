---
title: 告警处理
summary: 了解 DM 中各主要告警信息的处理方法。
aliases: ['/docs-cn/tidb-data-migration/stable/handle-alerts/','/docs-cn/tidb-data-migration/v1.0/handle-alerts/']
---

# 告警处理

本文档介绍 DM 中各主要告警信息的处理方法。

## 任务状态告警

### `DM_task_state`

当 DM-worker 内有子任务处于 `Paused` 状态超过 20 分钟时会触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

## relay log 告警

### `DM_relay_process_exits_with_error`

当 relay log 处理单元遇到错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_remain_storage_of_relay_log`

当 relay log 所在磁盘的剩余可用容量小于 10G 时会触发该告警，对应的处理方法包括：

- 手动清理该磁盘上其他无用数据以增加可用容量。
- 尝试调整 relay log 的[自动清理策略](relay-log.md#自动数据清理)或执行[手动清理](relay-log.md#手动数据清理)。
- 尝试[迁移 DM-worker 实例](cluster-operations.md#替换迁移-dm-worker-实例)到其他可用容量充足的磁盘上。

### `DM_relay_log_data_corruption`

当 relay log 处理单元在校验从上游读取到的 binlog event 且发现 checksum 信息异常时会转为 `Paused` 状态并立即触发告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_fail_to_read_binlog_from_master`

当 relay log 处理单元在尝试从上游读取 binlog event 发生错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_fail_to_write_relay_log`

当 relay log 处理单元在尝试将 binlog event 写入 relay log 文件发生错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_binlog_file_gap_between_master_relay`

当 relay log 处理单元已拉取到的最新的 binlog 文件个数落后于当前上游 MySQL/MariaDB 超过 1 个（不含 1 个）且持续 10 分钟时会触发该告警，此时需要参考[性能问题及处理方法](handle-performance-issues.md)对 relay log 处理单元相关的性能问题进行排查与处理。

## Dump/Load 告警

### `DM_dump_process_exists_with_error`

当 Dump 处理单元遇到错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_load_process_exists_with_error`

当 Load 处理单元遇到错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

## Binlog replication 告警

### `DM_sync_process_exists_with_error`

当 Binlog replication 处理单元遇到错误时，会转为 `Paused` 状态并立即触发该告警，此时需要参考 [DM 故障诊断](error-handling.md#dm-故障诊断)进行处理。

### `DM_binlog_file_gap_between_master_syncer`

当 Binlog replication 处理单元已处理到的最新的 binlog 文件个数落后于当前上游 MySQL/MariaDB 超过 1 个（不含 1 个）且持续 10 分钟时 DM 会触发该告警，此时需要参考[性能问题及处理方法](handle-performance-issues.md)对 Binlog replication 处理单元相关的性能问题进行排查与处理。

### `DM_binlog_file_gap_between_relay_syncer`

当 Binlog replication 处理单元已处理到的最新的 binlog 文件个数落后于当前 relay log 处理单元超过 1 个（不含 1 个）且持续 10 分钟时 DM 会触发该告警，此时需要参考[性能问题及处理方法](handle-performance-issues.md)对 Binlog replication 处理单元相关的性能问题进行排查与处理。
