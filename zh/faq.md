---
title: Data Migration 常见问题
aliases: ['/docs-cn/tidb-data-migration/dev/faq/']
---

# Data Migration 常见问题

## DM 是否支持迁移阿里 RDS 以及其他云数据库的数据？

DM 仅支持解析标准版本的 MySQL/MariaDB 的 binlog，对于阿里云 RDS 以及其他云数据库没有进行过测试，如果确认其 binlog 为标准格式，则可以支持。

## task 配置中的黑白名单的正则表达式是否支持`非获取匹配`（?!）？

目前不支持，DM 仅支持 golang 标准库的正则，可以通过 [re2-syntax](https://github.com/google/re2/wiki/Syntax) 了解 golang 支持的正则表达式。

## 如果在上游执行的一个 statement 包含多个 DDL 操作，DM 是否支持迁移？

DM 会尝试将包含多个 DDL 变更操作的单条语句拆分成只包含一个 DDL 操作的多条语句，但是可能没有覆盖所有的场景。建议在上游执行的一条 statement 中只包含一个 DDL 操作，或者在测试环境中验证一下，如果不支持，可以给 DM 提 [issue](https://github.com/pingcap/dm/issues)。

## 如何处理不兼容的 DDL 语句？

你需要使用 dmctl 手动处理 TiDB 不兼容的 DDL 语句（包括手动跳过该 DDL 语句或使用用户指定的 DDL 语句替换原 DDL 语句，详见[处理出错的 SQL 语句](handle-failed-sql-statements.md)）。

> **注意：**
>
> TiDB 目前并不兼容 MySQL 支持的所有 DDL 语句。

## 如何重置数据迁移任务？

当数据迁移过程中发生异常且无法恢复时，需要重置数据迁移任务，对数据重新进行迁移：

1. 使用 `stop-task` 停止异常的数据迁移任务。

2. 清理下游已迁移的数据。

3. 从下面两种方式中选择其中一种重启数据迁移任务：

    - 修改任务配置文件以指定新的任务名，然后使用 `start-task {task-config-file}` 重启迁移任务。
    - 使用 `start-task --remove-meta {task-config-file}` 重启数据迁移任务。

## 设置了 `online-ddl-scheme: "gh-ost"`， gh-ost 表相关的 DDL 报错该如何处理？

```
[unit=Sync] ["error information"="{\"msg\":\"[code=36046:class=sync-unit:scope=internal:level=high] online ddls on ghost table `xxx`.`_xxxx_gho`\\ngithub.com/pingcap/dm/pkg/terror.(*Error).Generate ......
```

出现上述错误可能有以下原因：

DM 在最后 `rename ghost_table to origin table` 的步骤会把内存的 DDL 信息读出，并且还原为 origin table 的 DDL。而内存中的 DDL 信息是在 `alter ghost_table` 的时候进行[处理](feature-online-ddl-scheme.md#online-schema-change-gh-ost)，记录 ghost_table DDL 的信息；或者是在重启 dm-worker 启动 task 的时候，从 `dm_meta.{task_name}_onlineddl` 中读取出来。

因此，如果在增量复制过程中，指定的 Pos 跳过了 `alter ghost_table` 的 DDL，但是该 Pos 仍在 gh-ost 的 online-ddl 的过程中，就会因为 ghost_table 没有正确写入到内存以及 `dm_meta.{task_name}_onlineddl`，而导致该问题。

可以通过以下方式绕过这个问题：

1. 取消 task 的 `online-ddl-schema` 的配置。

2. 把 `_{table_name}_gho`、`_{table_name}_ghc`、`_{table_name}_del` 配置到 `block-allow-list.ignore-tables` 中。

3. 手工在下游的 TiDB 执行上游的 DDL。

4. 待 Pos 复制到 gh-ost 整体流程后的位置，再重新启用 `online-ddl-schema` 以及注释掉 `block-allow-list.ignore-tables`。

## 如何为已有迁移任务增加需要迁移的表？

假如已有数据迁移任务正在运行，但又有其他的表需要添加到该迁移任务中，可根据当前数据迁移任务所处的阶段按下列方式分别进行处理。

> **注意：**
>
> 向已有数据迁移任务中增加需要迁移的表操作较复杂，请仅在确有强烈需求时进行。

### 迁移任务当前处于 `Dump` 阶段

由于 MySQL 不支持指定 snapshot 来进行导出，因此在导出过程中不支持更新迁移任务并重启以通过断点继续导出，故无法支持在该阶段动态增加需要迁移的表。

如果确实需要增加其他的表用于迁移，建议直接使用新的配置文件重新启动迁移任务。

### 迁移任务当前处于 `Load` 阶段

多个不同的数据迁移任务在导出时，通常对应于不同的 binlog position，如将它们在 `Load` 阶段合并导入，则无法就 binlog position 达成一致，因此不建议在 `Load` 阶段向数据迁移任务中增加需要迁移的表。

### 迁移任务当前处于 `Sync` 阶段

当数据迁移任务已经处于 `Sync` 阶段时，在配置文件中增加额外的表并重启任务，DM 并不会为新增的表重新执行全量导出与导入，而是会继续从之前的断点进行增量复制。

因此，如果需要新增的表对应的全量数据尚未导入到下游，则需要先使用单独的数据迁移任务将其全量数据导出并导入到下游。

将已有迁移任务对应的全局 checkpoint （`is_global=1`）中的 position 信息记为 `checkpoint-T`，如 `(mysql-bin.000100, 1234)`。将需要增加到迁移任务的表在全量导出的 `metedata`（或另一个处于 `Sync` 阶段的数据迁移任务的 checkpoint）的 position 信息记为 `checkpoint-S`，如 `(mysql-bin.000099, 5678)`。则可通过以下步骤将表增加到迁移任务中：

1. 使用 `stop-task` 停止已有迁移任务。如果需要增加的表属于另一个运行中的迁移任务，则也将其停止。

2. 使用 MySQL 客户连接到下游 TiDB 数据库，手动更新已有迁移任务对应的 checkpoint 表中的信息为 `checkpoint-T` 与 `checkpoint-S` 中的较小值（在本例中，为 `(mysql-bin.000099, 5678)`）。

    - 需要更新的 checkpoint 表为 `{dm_meta}` 库中的 `{task-name}_syncer_checkpoint`。

    - 需要更新的 checkpoint 行为 `id={source-id}` 且 `is_global=1`。
    
    - 需要更新的 checkpoint 列为 `binlog_name` 与 `binlog_pos`。

3. 在迁移任务配置中为 `syncers` 部分设置 `safe-mode: true` 以保证可重入执行。

4. 通过 `start-task` 启动迁移任务。

5. 通过 `query-status` 观察迁移任务状态，当 `syncerBinlog` 超过 `checkpoint-T` 与 `checkpoint-S` 中的较大值后（在本例中，为 `(mysql-bin.000100, 1234)`），即可还原 `safe-mode` 为原始值并重启迁移任务。

## 全量导入过程中遇到报错 `packet for query is too large. Try adjusting the 'max_allowed_packet' variable`

尝试将

- TiDB Server 的全局变量 `max_allowed_packet`
- 任务配置文件中的配置项 `target-database.max-allowed-packet`（详情参见 [DM 任务完整配置文件介绍](task-configuration-file-full.md)）

设置为比默认 67108864 (64M) 更大的值。详见 [Loader 解决方案](https://docs.pingcap.com/zh/tidb/stable/loader-overview#解决方案)。

## 2.0 集群运行 1.0 已有数据迁移任务时报错 `Error 1054: Unknown column 'binlog_gtid' in 'field list'`

在 DM 2.0 中，为 checkpoint 等元信息表引入了更多的字段。如果在 2.0 中，通过 `start-task` 直接使用 1.0 集群的任务配置文件从增量复制阶段继续运行，则会出现 `Error 1054: Unknown column 'binlog_gtid' in 'field list'` 错误。

对于此错误，可使用以下任一方式进行处理：

- [使用 TiUP 将 DM 1.0 集群导入到全新的 2.0 集群](maintain-dm-using-tiup.md#导入-dm-ansible-部署的-dm-10-集群并升级)。
- [手动将 DM 1.0 的数据迁移任务导入到 2.0 集群](manually-upgrade-dm-1.0-to-2.0.md)。

## TiUP 无法部署 DM v2.0.0-hotfix 版本

你可以通过 `tiup list dm-master` 命令查看 TiUP 支持部署的 DM 版本。该命令未展示的版本（如 v2.0.0-hotfix）不能由 TiUP 管理。

## DM 同步报错 `parse mydumper metadata error: EOF`

该错误需要查看报错信息以及日志进一步分析。报错原因可能是 dump 单元由于缺少权限没有产生正确的 metadata 文件。

## DM 分库分表同步中没有明显报错，但是下游数据丢失

需要检查配置项，block-allow-list 和 table-route。
block-allow-list 填写的是上游数据库表，可以在 do-tables 前通过加 “~” 来进行正则匹配。table-route 不支持正则，采用的是通配符模式，所以 table_parttern_[0-63] 只会匹配 table_parttern_0 到 table_pattern_6 这 7 张表。

## DM 上游无写入，replicate lag 监控无数据

在 v1.0 中，需要开启 `enable-heartbeat` 才会产生该监控项。v2.0 尚未启用该功能。
