---
title: Introduction to dmctl
summary: Learn how to manage the data replication task using dmctl.
aliases: ['/docs/tidb-data-migration/dev/manage-replication-tasks/','/tidb-data-migration/dev/manage-replication-tasks/']
---

# Introduction to dmctl

dmctl is a command line tool used to manage the data replication task. For DM clusters deployed using TiUP, you can directly use [`tiup dmctl`](maintain-dm-using-tiup.md#dmctl).

The dmctl component supports the interactive mode and the command mode.

## Interactive mode

Enter the interactive mode to interact with DM-master:

> **Note:**
>
> The interactive mode does not support Bash features. For example, you need to directly pass string flags instead of passing them in quotes.

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

## Command mode

The command mode differs from the interactive mode in that you need to append the task operation right after the dmctl command. The parameters of the task operation in the command mode are the same as those in the interactive mode.

> **Note:**
>
> + A dmctl command must be followed by only one task operation.
> + The task operation can be placed only at the end of the dmctl command.

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
  unlock-ddl-lock       unlock-ddl-lock <lock-ID>

Special Commands:
  --encrypt Encrypts plaintext to ciphertext.
  --decrypt Decrypts ciphertext to plaintext.

Global Options:
  --V Prints version and exit.
  --config Path to configuration file.
  --master-addr Master API server addr.
  --rpc-timeout RPC timeout, default is 10m.
  --ssl-ca Path of file that contains list of trusted SSL CAs for connection.
  --ssl-cert Path of file that contains X509 certificate in PEM format for connection.
  --ssl-key Path of file that contains X509 key in PEM format for connection.
```
