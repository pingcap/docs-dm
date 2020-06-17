---
title: Data Migration 常见问题
category: reference
aliases: ['/docs-cn/dev/faq/data-migration/','/docs-cn/dev/reference/tools/data-migration/faq/','/docs-cn/v3.1/reference/tools/data-migration/faq/','/docs-cn/v3.0/reference/tools/data-migration/faq/','/docs-cn/v2.1/reference/tools/data-migration/faq/']
---

# Data Migration 常见问题

## DM 是否支持同步阿里 RDS 以及其他云数据库的数据？

DM 仅支持解析标准版本的 MySQL/MariaDB 的 binlog，对于阿里云 RDS 以及其他云数据库没有进行过测试，如果确认其 binlog 为标准格式，则可以支持。

## task 配置中的黑白名单的正则表达式是否支持`非获取匹配`（?!）？

目前不支持，DM 仅支持 golang 标准库的正则，可以通过 [re2-syntax](https://github.com/google/re2/wiki/Syntax) 了解 golang 支持的正则表达式。

## 如果在上游执行的一个 statement 包含多个 DDL 操作，DM 是否支持同步？

DM 会尝试将包含多个 DDL 变更操作的单条语句拆分成只包含一个 DDL 操作的多条语句，但是可能没有覆盖所有的场景。建议在上游执行的一条 statement 中只包含一个 DDL 操作，或者在测试环境中验证一下，如果不支持，可以给 DM 提 [issue](https://github.com/pingcap/dm/issues)。

## 如何处理不兼容的 DDL 语句？

你需要使用 dmctl 手动处理 TiDB 不兼容的 DDL 语句（包括手动跳过该 DDL 语句或使用用户指定的 DDL 语句替换原 DDL 语句，详见[跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md)）。

> **注意：**
>
> TiDB 目前并不兼容 MySQL 支持的所有 DDL 语句。

## 如何重置数据同步任务？

### relay log 无异常时重置同步任务

如果数据同步任务所需要的 relay log 不存在异常，可使用如下步骤重置数据同步任务以对数据重新进行迁移：

1. 使用 `stop-task` 停止异常的数据迁移任务。

2. 清理掉下游已同步的数据。

3. 从下面两种方式中选择其中一种重启数据同步任务：

    - 修改任务配置文件以指定新的任务名，然后使用 `start-task` 重启同步任务。
    - 修改任务配置文件以设置 `remove-meta` 为 `true`，然后使用 `start-task` 重启同步任务。

### relay log 存在异常时重置同步任务

#### 所需的 relay log 在上游 MySQL 中存在

如果同步任务所需要的 relay log 在 DM-worker 中存在异常，但仍然正常存在于上游 MySQL 中时，可使用如下步骤恢复数据同步任务：

1. 使用 `stop-task` 停止当前正在运行的所有同步任务。

2. 参考[重启 DM-worker](cluster-operations.md#重启-dm-worker)文档，**停止**存在异常的 DM-worker 节点。

3. 从上游 MySQL 中复制正常的 binlog 文件以替换 DM-worker 的 [relay log 目录](relay-log.md#目录结构)内的对应文件。

    - 如果是使用 DM-Ansible 部署，relay log 目录即 `<deploy_dir>/relay_log` 目录。
    - 如果是使用二进制文件手动部署，relay log 目录即 `relay-dir` 参数设置的目录。

4. 修改 DM-worker 的 relay log 目录内的 `relay.meta` 的信息为下一个 binlog 文件对应的信息。

    - 如果未启用 `enable-gtid`，则将 `binlog-name` 设置为下一个 binlog 文件的文件名，并将 `binlog-pos` 设置为 `4`。如从上游 MySQL 复制了 `mysq-bin.000100` 到 relay 目录，并期望之后从 `mysql-bin.000101` 开始继续 binlog 的拉取，则将 `binlog-name` 设置为 `mysql-bin.000101`。
    - 如果启用了 `enable-gtid`，则将 `binlog-gtid` 设置为下一个 binlog 文件起始处的 `Previous_gtids` 对应的值（可通过在上游 MySQL 执行 [SHOW BINLOG EVENTS](https://dev.mysql.com/doc/refman/5.7/en/show-binlog-events.html) 获得）。

5. 参考[重启 DM-worker](cluster-operations.md#重启-dm-worker)文档，**启动**存在异常的 DM-worker 节点。

6. 使用 `start-task` 恢复已停止的同步任务。

#### 所需的 relay log 在上游 MySQL 已清理

如果同步任务所需要的 relay log 在 DM-worker 中存在异常、且在上游已被清理，可使用如下步骤重置数据同步任务以对数据重新进行迁移：

1. 使用 `stop-task` 命令停止当前正在运行的所有同步任务。

2. 使用 DM-Ansible [停止整个 DM 集群](deploy-a-dm-cluster-using-ansible.md#第-10-步关闭-dm-集群)。

3. 手动清理掉与 binlog event 被重置的 MySQL 相对应的 DM-worker 的 relay log 目录。

    - 如果是使用 DM-Ansible 部署，relay log 目录即 `<deploy_dir>/relay_log` 目录。
    - 如果是使用二进制文件手动部署，relay log 目录即 `relay-dir` 参数设置的目录。

4. 清理掉下游已同步的数据。

5. 使用 DM-Ansible [启动整个 DM 集群](deploy-a-dm-cluster-using-ansible.md#第-9-步部署-dm-集群)。

6. 从下面两种方式中选择其中一种重启数据同步任务：
   
    - 修改任务配置文件以指定新的任务名，然后使用 `start-task` 重启同步任务。
    - 修改任务配置文件以设置 `remove-meta` 为 `true`，然后使用 `start-task` 重启同步任务。

## 设置了 `online-ddl-scheme: "gh-ost"`， gh-ost 表相关的 DDL 报错该如何处理？

```
[unit=Sync] ["error information"="{\"msg\":\"[code=36046:class=sync-unit:scope=internal:level=high] online ddls on ghost table `xxx`.`_xxxx_gho`\\ngithub.com/pingcap/dm/pkg/terror.(*Error).Generate ......
```

出现上述错误可能有以下原因：

DM 在最后 `rename ghost_table to origin table` 的步骤会把内存的 DDL 信息读出，并且还原为 origin table 的 DDL。而内存中的 DDL 信息是在 `alter ghost_table` 的时候进行[处理](feature-online-ddl-scheme.md#dm-对于-online-ddl-scheme-gh-ost-的处理)，记录 ghost_table DDL 的信息；或者是在重启 dm-worker 启动 task 的时候，从 `dm_meta.{task_name}_onlineddl` 中读取出来。

因此，如果在增量同步过程中，指定的 Pos 跳过了 `alter ghost_table` 的 DDL，但是该 Pos 仍在 gh-ost 的 online-ddl 的过程中，就会因为 ghost_table 没有正确写入到内存以及 `dm_meta.{task_name}_onlineddl`，而导致该问题。

可以通过以下方式绕过这个问题：

1. 取消 task 的 `online-ddl-schema` 的配置。

2. 把 `_{table_name}_gho`、`_{table_name}_ghc`、`_{table_name}_del` 配置到 `black-white-list.ignore-tables` 中。

3. 手工在下游的 TiDB 执行上游的 DDL。

4. 待 Pos 同步到 gh-ost 整体流程后的位置，再重新启用 `online-ddl-schema` 以及注释掉 `black-white-list.ignore-tables`。
