---
title: dmctl 简介
summary: 了解如何使用 dmctl 管理数据同步任务。
category: reference
aliases: ['/docs-cn/tidb-data-migration/dev/manage-replication-tasks/']
---

# dmctl 简介

dmctl 是用来控制 DM 集群的命令行工具。对于用 DM-Ansible 部署的 DM 集群，dmctl 二进制文件路径为 `dm-ansible/dmctl`。

dmctl 同时支持交互模式和命令模式。

## dmctl 交互模式

进入交互模式，与 DM-master 进行交互：

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
  operate-source       create/update/stop upstream MySQL/MariaDB source
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
  update-task          update a task's config for routes, filters, or black-white-list

Flags:
  -h, --help             help for dmctl
  -s, --source strings   MySQL Source ID

Use `dmctl [command] --help` to get more information about a command.
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
  migrate-relay         migrate-relay <source> <binlogName> <binlogPos>
  offline-worker        offline-worker <name> <address>
  operate-source        operate-source <operate-type> <config-file>
  pause-relay           pause-relay <-s source ...>
  pause-task            pause-task [-s source ...] <task-name>
  purge-relay           purge-relay <-s source> [--filename] [--sub-dir]
  query-error           query-error [-s source ...] [task-name]
  query-status          query-status [-s source ...] [task-name] [--more]
  resume-relay          resume-relay <-s source ...>
  resume-task           resume-task [-s source ...] <task-name>
  show-ddl-locks        show-ddl-locks [-s source ...] [task-name]
  sql-inject            sql-inject <-s source> <task-name> <sql1;sql2;>
  sql-replace           sql-replace <-s source> [-b binlog-pos] [-p sql-pattern] [--sharding] <task-name> <sql1;sql2;>
  sql-skip              sql-skip <-s source> [-b binlog-pos] [-p sql-pattern] [--sharding] <task-name>
  start-task            start-task [-s source ...] <config-file>
  stop-task             stop-task [-s source ...] <task-name>
  switch-relay-master   switch-relay-master <-s source ...>
  unlock-ddl-lock       unlock-ddl-lock [-s source ...] <lock-ID>
  update-master-config  update-master-config <config-file>
  update-relay          update-relay [-s source ...] <config-file>
  update-task           update-task [-s source ...] <config-file>
```
