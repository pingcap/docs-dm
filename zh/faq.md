---
title: Data Migration 常见问题
aliases: ['/docs-cn/tidb-data-migration/stable/faq/','/docs-cn/tidb-data-migration/v1.0/faq/','/docs-cn/dev/faq/data-migration/','/docs-cn/dev/reference/tools/data-migration/faq/','/docs-cn/v3.1/reference/tools/data-migration/faq/','/docs-cn/v3.0/reference/tools/data-migration/faq/','/docs-cn/v2.1/reference/tools/data-migration/faq/','/docs-cn/v2.1/faq/data-migration/']
---

# Data Migration 常见问题

## DM 是否支持迁移阿里 RDS 以及其他云数据库的数据？

DM 仅支持解析标准版本的 MySQL/MariaDB 的 binlog，对于阿里云 RDS 以及其他云数据库没有进行过测试，如果确认其 binlog 为标准格式，则可以支持。

已知问题的兼容情况：

- 阿里云 RDS
    - 即使上游表没有主键，阿里云 RDS 的 binlog 中也会包含隐藏的主键列，与上游表结构不一致。
- 华为云 RDS
    - 不支持，详见：[华为云数据库 RDS 是否支持直接读取 Binlog 备份文件](https://support.huaweicloud.com/rds_faq/rds_faq_0210.html)。

## task 配置中的黑白名单的正则表达式是否支持`非获取匹配`（?!）？

目前不支持，DM 仅支持 golang 标准库的正则，可以通过 [re2-syntax](https://github.com/google/re2/wiki/Syntax) 了解 golang 支持的正则表达式。

## 如果在上游执行的一个 statement 包含多个 DDL 操作，DM 是否支持迁移？

DM 会尝试将包含多个 DDL 变更操作的单条语句拆分成只包含一个 DDL 操作的多条语句，但是可能没有覆盖所有的场景。建议在上游执行的一条 statement 中只包含一个 DDL 操作，或者在测试环境中验证一下，如果不支持，可以给 DM 提 [issue](https://github.com/pingcap/dm/issues)。

## 如何处理不兼容的 DDL 语句？

你需要使用 dmctl 手动处理 TiDB 不兼容的 DDL 语句（包括手动跳过该 DDL 语句或使用用户指定的 DDL 语句替换原 DDL 语句，详见[跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)）。

> **注意：**
>
> TiDB 目前并不兼容 MySQL 支持的所有 DDL 语句。

## 如何重置数据迁移任务？

### relay log 无异常时重置迁移任务

如果数据迁移任务所需要的 relay log 不存在异常，可使用如下步骤重置数据迁移任务以对数据重新进行迁移：

1. 使用 `stop-task` 停止异常的数据迁移任务。

2. 清理下游已迁移的数据。

3. 从下面两种方式中选择其中一种重启数据迁移任务：

    - 修改任务配置文件以指定新的任务名，然后使用 `start-task` 重启迁移任务。
    - 修改任务配置文件以设置 `remove-meta` 为 `true`，然后使用 `start-task` 重启迁移任务。

### relay log 存在异常时重置迁移任务

#### 所需的 relay log 在上游 MySQL 中存在

如果迁移任务所需要的 relay log 在 DM-worker 中存在异常，但仍然正常存在于上游 MySQL 中时，可使用如下步骤恢复数据迁移任务：

1. 使用 `stop-task` 停止当前正在运行的所有迁移任务。

2. 参考[重启 DM-worker](cluster-operations.md#重启-dm-worker)文档，**停止**存在异常的 DM-worker 节点。

3. 从上游 MySQL 中复制正常的 binlog 文件以替换 DM-worker 的 [relay log 目录](relay-log.md#目录结构)内的对应文件。

    - 如果是使用 DM-Ansible 部署，relay log 目录即 `<deploy_dir>/relay_log` 目录。
    - 如果是使用二进制文件手动部署，relay log 目录即 `relay-dir` 参数设置的目录。

4. 修改 DM-worker 的 relay log 目录内的 `relay.meta` 的信息为下一个 binlog 文件对应的信息。

    - 如果未启用 `enable-gtid`，则将 `binlog-name` 设置为下一个 binlog 文件的文件名，并将 `binlog-pos` 设置为 `4`。如从上游 MySQL 复制了 `mysq-bin.000100` 到 relay 目录，并期望之后从 `mysql-bin.000101` 开始继续 binlog 的拉取，则将 `binlog-name` 设置为 `mysql-bin.000101`。
    - 如果启用了 `enable-gtid`，则将 `binlog-gtid` 设置为下一个 binlog 文件起始处的 `Previous_gtids` 对应的值（可通过在上游 MySQL 执行 [SHOW BINLOG EVENTS](https://dev.mysql.com/doc/refman/5.7/en/show-binlog-events.html) 获得）。

5. 参考[重启 DM-worker](cluster-operations.md#重启-dm-worker)文档，**启动**存在异常的 DM-worker 节点。

6. 使用 `start-task` 恢复已停止的迁移任务。

#### 所需的 relay log 在上游 MySQL 已清理

如果迁移任务所需要的 relay log 在 DM-worker 中存在异常、且在上游已被清理，可使用如下步骤重置数据迁移任务以对数据重新进行迁移：

1. 使用 `stop-task` 命令停止当前正在运行的所有迁移任务。

2. 使用 DM-Ansible [停止整个 DM 集群](deploy-a-dm-cluster-using-ansible.md#第-10-步关闭-dm-集群)。

3. 手动清理掉与 binlog event 被重置的 MySQL 相对应的 DM-worker 的 relay log 目录。

    - 如果是使用 DM-Ansible 部署，relay log 目录即 `<deploy_dir>/relay_log` 目录。
    - 如果是使用二进制文件手动部署，relay log 目录即 `relay-dir` 参数设置的目录。

4. 清理掉下游已迁移的数据。

5. 使用 DM-Ansible [启动整个 DM 集群](deploy-a-dm-cluster-using-ansible.md#第-9-步部署-dm-集群)。

6. 从下面两种方式中选择其中一种重启数据迁移任务：

    - 修改任务配置文件以指定新的任务名，然后使用 `start-task` 重启迁移任务。
    - 修改任务配置文件以设置 `remove-meta` 为 `true`，然后使用 `start-task` 重启迁移任务。

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

## DM v1.0 在任务出错时使用 `sql-skip` 命令无法跳过某些语句

首先需要检查执行 `sql-skip` 之后 binlog 位置是否在推进，如果是的话表示 `sql-skip` 已经生效。重复出错的原因是上游发送了多个不支持的 DDL，可以通过 `sql-skip -s <sql-pattern>` 进行模式匹配。

对于类似下面这种报错（报错中包含 `parse statement`）：

```
if the DDL is not needed, you can use a filter rule with \"*\" schema-pattern to ignore it.\n\t : parse statement: line 1 column 11 near \"EVENT `event_del_big_table` \r\nDISABLE\" %!!(MISSING)(EXTRA string=ALTER EVENT `event_del_big_table` \r\nDISABLE
```

出现报错的原因是 TiDB parser 无法解析上游的 DDL，例如 `ALTER EVENT`，所以 `sql-skip` 不会按预期生效。可以在任务配置文件中添加 [Binlog 过滤规则](feature-overview.md#binlog-event-filter)进行过滤，并设置 `schema-pattern: "*"`。

## DM 同步时下游长时间出现 REPLACE 语句

请检查是否符合 [safe mode 触发条件](glossary.md#safe-mode)。如果任务发生错误并自动恢复，会满足“启动或恢复任务的前 5 分钟“这一条件，因此启用 safe mode。

可以检查 DM-worker 日志，在其中搜索包含 `change count` 的行，该行的 `new count` 非零时会启用 safe mode。检查 safe mode 启用时间以及启用前是否有报错，以定位启用原因。

## 使用 dmctl 执行命令时无法连接 DM-master

在使用 dmctl 执行相关命令时，发现连接 DM-master 失败（即使已在命令中指定 `--master-addr` 的参数值），报错内容类似 `RawCause: context deadline exceeded, Workaround: please check your network connection.`，但使用 `telnet <master-addr>` 之类的命令检查网络却没有发现异常。  

这种情况可以检查下环境变量 `https_proxy`（注意，这里是 **https** ）。如果配置了该环境变量，dmctl 会自动去连接 `https_proxy`  指定的主机及端口，而如果该主机没有相应的 `proxy` 转发服务，则会导致连接失败。  

解决方案：确认 `https_proxy` 是否必须要配置，如果不是必须的，取消该设置即可。如果环境必须，那么在原命令前加环境变量设置 `https_proxy="" ./dmctl --master-addr "x.x.x.x:8261"` 即可。

> **注意：**   
> 
> 关于 `proxy` 的环境变量有 `http_proxy`，`https_proxy`，`no_proxy` 等。如果依据上述解决方案处理后仍无法连接，可以考虑检查 `http_proxy` 和 `no_proxy` 的参数配置是否有影响。 