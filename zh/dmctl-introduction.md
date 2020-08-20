---
title: dmctl 简介
summary: 了解如何使用 dmctl 管理数据同步任务。
aliases: ['/docs-cn/tidb-data-migration/dev/dmctl-introduction/','/docs-cn/tidb-data-migration/dev/manage-replication-tasks/']
---

# dmctl 简介

dmctl 是用来控制 DM 集群的命令行工具。对于用 TiUP 部署的 DM 集群，可以直接使用 [`tiup dmctl`](maintain-dm-using-tiup.md#集群控制工具-dmctl)。

dmctl 同时支持交互模式和命令模式。

## dmctl 交互模式

进入交互模式，与 DM-master 进行交互：

> **注意：**
>
> 交互模式下不具有 bash 的特性，比如不需要通过引号传递字符串参数而应当直接传递。

{{< copyable "shell-regular" >}}

```bash
./dmctl -master-addr 172.16.30.14:8261
```

```
Welcome to dmctl
Release Version: v1.0.1
Git Commit Hash: e63c6cdebea0edcf2ef8c91d84cff4aaa5fc2df7
Git Branch: release-1.0
UTC Build Time: 2019-09-10 06:15:05
Go Version: go version go1.12 linux/amd64

» help
DM control

Usage:
  dmctl [command]

Available Commands:
  check-task      Checks the configuration file of the task.
  get-task-config Gets the task configuration.
  handle-error    skip/replace/revert the current error event or a specific binlog position (binlog-pos) event.
  help            Help about any command.
  list-member     Lists member information.
  offline-member  Offlines member which has been closed.
  operate-leader  evict/cancel-evict the leader.
  operate-schema  get/set/remove the schema for an upstream table.
  operate-source  create/update/stop/show upstream MySQL/MariaDB source.
  pause-task      Pauses a specified running task.
  query-status    Queries task status.
  resume-task     Resumes a specified paused task.
  show-ddl-locks  Shows un-resolved DDL locks.
  start-task      Starts a task as defined in the configuration file.
  stop-task       Stops a specified task.
  unlock-ddl-lock Unlocks DDL lock forcefully.

Flags:
  -h, --help             Help for dmctl.
  -s, --source strings   MySQL Source ID.

Use "dmctl [command] --help" for more information about a command.
```

## dmctl 命令模式

命令模式跟交互模式的区别是，执行命令时只需要在 dmctl 命令后紧接着执行任务操作，任务操作同交互模式的参数一致。

> **注意：**
>
> + 一条 dmctl 命令只能跟一个任务操作
> + 任务操作只能放在 dmctl 命令的最后

{{< copyable "shell-regular" >}}

```bash
./dmctl -master-addr 172.16.30.14:8261 start-task task.yaml
./dmctl -master-addr 172.16.30.14:8261 stop-task task
./dmctl -master-addr 172.16.30.14:8261 query-status
```

```
Available Commands:
  check-task            check-task <config-file>
  get-task-config       get-task-config <task-name | task-file> [--file filename]
  handle-error          handle-error <task-name | task-file> [-s source ...] [-b binlog-pos] <skip/replace/revert> [replace-sql1;replace-sql2;]
  list-member           list-member [--leader] [--master] [--worker] [--name master-name/worker-name ...]
  offline-member        offline-member <--master/--worker> <--name master-name/worker-name>
  operate-leader        operate-leader <operate-type>
  operate-schema        operate-schema <operate-type> <-s source ...> <task-name | task-file> <-d database> <-t table> [schema-file]
  operate-source        operate-source <operate-type> [config-file ...] [--print-sample-config]
  pause-task            pause-task [-s source ...] <task-name | task-file>
  query-status          query-status [-s source ...] [task-name] [--more]
  resume-task           resume-task [-s source ...] <task-name | task-file>
  show-ddl-locks        show-ddl-locks [-s source ...] [task-name]
  start-task            start-task [-s source ...] [--remove-meta] <config-file>
  stop-task             stop-task [-s source ...] <task-name | task-file>
  unlock-ddl-lock       unlock-ddl-lock [-s source ...] <lock-ID>

Special Commands:
  --encrypt Encrypts plaintext to ciphertext.
  --decrypt Decrypts ciphertext to plaintext.

Global Options:
  --V Prints version and exit.
  --config Path to configuration file.
  --master-addr Master API server address.
  --rpc-timeout RPC timeout, default is 10m.
  --ssl-ca Path of file that contains list of trusted SSL CAs for connection.
  --ssl-cert Path of file that contains X509 certificate in PEM format for connection.
  --ssl-key Path of file that contains X509 key in PEM format for connection.
```
