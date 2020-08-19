---
title: Introduction to dmctl
summary: Learn how to manage the data replication task using dmctl.
aliases: ['/docs/tidb-data-migration/dev/manage-replication-tasks/','/tidb-data-migration/dev/manage-replication-tasks/']
---

# Introduction to dmctl

dmctl is a command line tool used to manage the data replication task. For DM clusters deployed using DM-Ansible, the binary file path of dmctl is `dm-ansible/dmctl`.

The dmctl component supports the interactive mode and the command mode.

## Interactive mode

Enter the interactive mode to interact with DM-master:

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
  check-task           check the config file of the task
  help                 help about any command
  migrate-relay        migrate DM-worker's relay unit
  offline-worker       offline worker which has been closed
  operate-source       create/update/stop/show upstream MySQL/MariaDB source
  pause-relay          pause DM-worker's relay unit
  pause-task           pause a specified running task
  purge-relay          purge relay log files of the DM-worker according to the specified filename
  query-error          query task error
  query-status         query task status
  resume-relay         resume DM-worker's relay unit
  resume-task          resume a specified paused task
  show-ddl-locks       show un-resolved DDL locks
  sql-inject           inject (limited) SQLs into binlog replication unit as binlog events
  sql-replace          replace SQLs matched by a specific binlog position (binlog-pos) or a SQL pattern (sql-pattern); each SQL must end with a semicolon
  sql-skip             skip the binlog event matched by a specific binlog position (binlog-pos) or a SQL pattern (sql-pattern)
  start-task           start a task as defined in the config file
  stop-task            stop a specified task
  switch-relay-master  switch the master server of the DM-worker's relay unit
  unlock-ddl-lock      forcefully unlock DDL lock
  update-master-config update the config of the DM-master
  update-relay         update the relay unit config of the DM-worker
  update-task          update a task's config for routes, filters, or block-allow-list

Flags:
  -h, --help             help for dmctl
  -s, --source strings   MySQL Source ID

Use `dmctl [command] --help` to get more information about a command.
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
  migrate-relay         migrate-relay <source> <binlogName> <binlogPos>
  offline-worker        offline-worker <name> <address>
  operate-source        operate-source <operate-type> [config-file ...] [--print-sample-config]
  pause-relay           pause-relay <-s source ...>
  pause-task            pause-task [-s source ...] <task-name | task-file>
  purge-relay           purge-relay <-s source> [--filename] [--sub-dir]
  query-error           query-error [-s source ...] [task-name]
  query-status          query-status [-s source ...] [task-name] [--more]
  resume-relay          resume-relay <-s source ...>
  resume-task           resume-task [-s source ...] <task-name | task-file>
  show-ddl-locks        show-ddl-locks [-s source ...] [task-name]
  sql-inject            sql-inject <-s source> <task-name> <sql1;sql2;>
  sql-replace           sql-replace <-s source> [-b binlog-pos] [-p sql-pattern] [--sharding] <task-name> <sql1;sql2;>
  sql-skip              sql-skip <-s source> [-b binlog-pos] [-p sql-pattern] [--sharding] <task-name>
  start-task            start-task [-s source ...] <config-file>
  stop-task             stop-task [-s source ...] <task-name | task-file>
  switch-relay-master   switch-relay-master <-s source ...>
  unlock-ddl-lock       unlock-ddl-lock [-s source ...] <lock-ID>
  update-master-config  update-master-config <config-file>
  update-relay          update-relay [-s source ...] <config-file>
  update-task           update-task [-s source ...] <config-file>
```