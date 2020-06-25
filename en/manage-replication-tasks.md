---
title: Manage the Data Replication Task
summary: Use dmctl to manage the data replication task.
category: reference
aliases: ['/docs/tidb-data-migration/dev/manage-replication-tasks/']
---

# Manage the Data Replication Task

This document describes how to manage and maintain the data replication task using the [dmctl](overview.md#dmctl) component. For the Data Migration cluster deployed using DM-Ansible, the dmctl binary file is in `dm-ansible/dmctl`.

The dmctl component supports the interactive mode for manual operations, and also supports the command mode for the script.

## dmctl interactive mode

This section describes the basic use of dmctl commands in the interactive mode.

### dmctl help

{{< copyable "shell-regular" >}}

```bash
./dmctl --help
```

```
Usage: dmctl [global options] command [command options] [arguments...]

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
Special Commands:
  --encrypt encrypt plaintext to ciphertext
Global Options:
  --V prints version and exit
  --config path to config file
  --master-addr master API server addr
  --rpc-timeout rpc timeout; default value is 10m
```

### Database password encryption

In DM configuration files, you need to use the password encrypted using dmctl, otherwise an error occurs. For a same original password, the password is different after each encryption.

```bash
$ ./dmctl -encrypt 123456
VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=
```

### Task management overview

Enter the interactive mode to interact with DM-master.

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

## Manage the data replication task

This section describes how to use the task management commands to execute corresponding operations.

### Load the data source configuration

You can use the `operate-source` command to load the data source configuration to the DM cluster.

{{< copyable "" >}}

```bash
help operate-source
```

```
create/update/stop upstream MySQL/MariaDB source
Usage:
  dmctl operate-source <operate-type> <config-file> [flags]
Flags:
  -h, --help   help for operate-source
Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
operate-source create ./source.toml
```

#### Flags description

+ `create`: Creates an upstream database source.

+ `update`: Updates an upstream database source.

+ `stop`: Stops an upstream database source.

+ `config-file`: (Required) Specifies the file path of `source.toml`.

#### Returned results

{{< copyable "" >}}

```bash
operate-source create ./source.toml
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

### Create the data replication task

You can use the `start-task` command to create the data replication task. Data Migration [prechecks the corresponding privileges and configuration automatically](precheck.md) while starting the data replication.

{{< copyable "" >}}

```bash
help start-task
```

```
start a task as defined in the config file

Usage:
 dmctl start-task [-s source ...] <config-file> [flags]

Flags:
 -h, --help   help for start-task

Global Flags:
 -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
start-task [ -s "mysql-replica-01"] ./task.yaml
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source to execute `task.yaml`. If it is set, only subtasks of the specified task on the MySQL source are started.
- `config-file`: (Required) Specifies the file path of `task.yaml`.

#### Returned results

{{< copyable "" >}}

```bash
start-task task.yaml
```

```bash
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Check the data replication task status

You can use the `query-status` task management command to check the status of the data replication task. For details about the query result and subtask status, see [Query Status](query-status.md).

```bash
help query-status
```

```
query task status

Usage:
 dmctl query-status [-s source ...] [task-name] [--more] [flags]

Flags:
 -h, --help   help for query-status
     --more   whether to print the detailed task information

Global Flags:
 -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
query-status
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to query) run.
- `task-name`: (Optional) Specifies the task name. If it is not set, the results of all data replication tasks are returned.

#### Returned results

For detailed description of query parameters and a complete list of returned result, refer to [Query status](query-status.md#query-result).

### Check query errors

You can use `query-error` to check error information on replication tasks or relay units. Compared to `query-status`, `query-error` only retrieves information related to the error itself.

`query-error` is often used to obtain the binlog position information required by `sql-skip`/`sql-replace`. For details on the flags and results of `query-error`, refer to [`query-error` in Skip or Replace Abnormal SQL Statements](skip-or-replace-abnormal-sql-statements.md#query-error).

### Pause the data replication task

You can use the `pause-task` command to pause a data replication task.

{{< copyable "" >}}

```bash
help pause-task
```

```
pause a specified running task

Usage:
 dmctl pause-task [-s source ...] <task-name> [flags]

Flags:
 -h, --help   help for pause-task

Global Flags:
 -s, --source strings   MySQL Source ID
```

> **Note:**
>
> The differences between `pause-task` and `stop-task` are:
>
> - `pause-task` only pauses a replication task, and the task information is retained in the memory, so that you can query using `query-status`. `stop-task` terminates a replication task and removes all task related information from the memory. This means you cannot use `query-status` to query. Data and the corresponding `dm_meta` like "checkpoint" that have been replicated to the downstream are not affected.
>
> - `pause-task` is generally used to pause the task for troubleshooting, while `stop-task` is used to permanently end a replication task, or co-work with `start-task` to update the configuration information.

#### Command usage example

{{< copyable "" >}}

```bash
pause-task [-s "mysql-replica-01"] task-name
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to pause) run. If it is set, only subtasks on the specified MySQL source are paused.
- `task-name`: (Required) Specifies the task name.

#### Returned results

```bash
pause-task test
```

```
{
​    "op": "Pause",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Resume the data replication task

You can use the `resume-task` command to resume the data replication task in the `Paused` state. This is generally used in scenarios where you want to manually resume a data replication task after you handle the errors that cause the task to pause.

```bash
help resume-task
```

```
resume a specified paused task

Usage:
 dmctl resume-task [-s source ...] <task-name> [flags]

Flags:
 -h, --help   help for resume-task

Global Flags:
 -s, --source strings   MySQL Source ID
```

#### Command usage example

```bash
resume-task [-s "mysql-replica-01"] task-name
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to restart) run. If it is set, only subtasks on the specified MySQL source are restarted.
- `task-name`: (Required) Specifies the task name.

#### Returned results

```bash
resume-task test
```

```bash
{
    "op": "Resume",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Stop the data replication task

You can use the `stop-task` command to stop a data replication task. For differences between `stop-task` and `pause-task`, refer to [Pause the data replication task](#pause-the-data-replication-task).

```bash
help stop-task
```

```
stop a specified task

Usage:
 dmctl stop-task [-s source ...] <task-name> [flags]

Flags:
 -h, --help   help for stop-task

Global Flags:
 -s, --source strings   MySQL Source ID
```

#### Command usage example

```bash
stop-task [-s "mysql-replica-01"]  task-name
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to stop) run. If it is set, only subtasks on the specified MySQL source are stopped.
- `task-name`: (Required) Specifies the task name.

#### Returned results

```bash
stop-task test
```

```
{
    "op": "Stop",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Update the data replication task

You can use the `update-task` command to update the data replication task. The following items support online update, while all other items do not support online update.

- table route rules
- black white list
- binlog filter rules

> **Note:**
>
> If you can make sure that the relay log required by the replication task will not be removed when the task is stopped, it is recommended that you use [Update items that do not support online update](#update-items-that-do-not-support-online-update) to update task configurations.

#### Update items that support online update

1. Check the status of the corresponding data replication task using `query-status <task-name>`.

    If `stage` is not `Paused`, use `pause-task <task-name>` to pause the task.

2. Edit the `task.yaml` file to update the custom configuration that you need to modify and the incorrect configuration.

3. Update the task configuration using `update-task task.yaml`.

4. Resume the task using `resume-task <task-name>`.

#### Update items that do not support online update

1. Check the status of the corresponding data replication task using `query-status <task-name>`.

    If the task exists, use `stop-task <task-name>` to stop the task.

2. Edit the `task.yaml` file to update the custom configuration that you need to modify and the incorrect configuration.

3. Restart the task using `start-task <task-name>`.

#### Command usage help

```bash
help update-task
```

```
update a task's config for routes, filters, black-white-list

Usage:
  dmctl update-task [-s source ...] <config-file> [flags]

Flags:
  -h, --help   help for update-task

Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

```bash
update-task [-s "mysql-replica-01"] ./task.yaml
```

#### Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to update) run. If it is set, only subtasks on the specified MySQL source are updated.
- `config-file`: (Required) Specifies the file path of `task.yaml`.

#### Returned results

```bash
update-task task_all_black.yaml
```

```bash
{
​    "result": true,
    "msg": "",
    "sources": [
    ]
}
```

## Manage DDL locks

Currently, DDL lock related commands mainly include `show-ddl-locks`, `unlock-ddl-lock`. For more information on their functions, usages, and applicable scenarios, refer to [Handle Sharding DDL Locks Manually](feature-manually-handling-sharding-ddl-locks.md).

## Other task and cluster management commands

In addition to the common task management commands above, DM also provides some other commands to manage data replication tasks and DM clusters.

### Check the task configuration file

You can use the `check-task` command to check whether a specified configuration file (`task.yaml`) of the replication task is valid, or whether the configuration of upstream/downstream database, permission setting, and schema meet the replication requirements. For more details, refer to [Precheck the upstream MySQL instance configuration](precheck.md).

When you use `start-task` to start a replication task, DM also executes all checks done by `check-task`.

{{< copyable "" >}}

```bash
help check-task
```

```
check the config file of the task

Usage:
 dmctl check-task <config-file> [flags]

Flags:
 -h, --help   help for check-task

Global Flags:
 -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
check-task task.yaml
```

#### Flags description

+ `config-file`: (Required) Specifies the path of the `task.yaml` file

#### Returned results

{{< copyable "" >}}

```bash
check-task task-test.yaml
```

```
{
    "result": true,
    "msg": "check pass!!!"
}
```

### Pause a relay unit

Relay units automatically run after the DM-worker thread starts. You can use the `pause-relay` command to pause the running relay units.

When you want to switch the DM-worker to connect to an upstream MySQL via a virtual IP, use `pause-relay` to make corresponding changes on DM.

{{< copyable "" >}}

```bash
help pause-relay
```

```
pause DM-worker's relay unit

Usage:
  dmctl pause-relay <-s source ...> [flags]

Flags:
  -h, --help   help for pause-relay

Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
pause-relay -s "mysql-replica-01"
```

#### Flags description

- `-s`: (Required) Specifies the MySQL source for which to pause the relay unit

#### Returned results

{{< copyable "" >}}

```bash
pause-relay -s "mysql-replica-01"
```

```
{
    "op": "ResumeRelay",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Resume a relay unit

You can use the `resume-relay` command to resume a relay unit in `Paused` state.

When you want to switch the DM-worker to connect to an upstream MySQL via a virtual IP, use `resume-relay` to make corresponding changes on DM.

{{< copyable "" >}}

```bash
help resume-relay
```

```
resume DM-worker's relay unit

Usage:
  dmctl resume-relay <-s source ...> [flags]

Flags:
  -h, --help   help for resume-relay

Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
resume-relay -s "mysql-replica-01"
```

#### Flags description

- `-s`: (Required) Specifies the MySQL source for which to resume the relay unit

#### Returned results

{{< copyable "" >}}

```bash
resume-relay -s "mysql-replica-01"
```

```
{
    "op": "ResumeRelay",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

### Switch the sub-directory for relay logs

Relay units store the binlog data from upstream MySQL instances in sub-directories. You can use the `switch-relay-master` command to swith the relay unit to use a new sub-directory.

When you want to switch the DM-worker to connect to an upstream MySQL via a virtual IP, use `switch-relay-master` to make corresponding changes on DM.

{{< copyable "" >}}

```bash
help switch-relay-master
```

```
switch the master server of the DM-worker's relay unit

Usage:
  dmctl switch-relay-master <-s source ...> [flags]

Flags:
  -h, --help   help for switch-relay-master

Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
switch-relay-master -s "mysql-replica-01"
```

#### Flags description

- `-s`: (Required) Specifies the MySQL source for which to switch the relay unit

#### Returned results

{{< copyable "" >}}

```bash
switch-relay-master -s "mysql-replica-01"
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": ""
        }
    ]
}
```

### Manually purge relay log

DM supports [Automatic data purge](relay-log.md#automatic-data-purge). You can also use `purge-relay` to [manually purge data](relay-log.md#manual-data-purge).

{{< copyable "" >}}

```bash
help purge-relay
```

```
purge relay log files of the DM-worker according to the specified filename

Usage:
  dmctl purge-relay <-s source> [--filename] [--sub-dir] [flags]

Flags:
  -f, --filename string   name of the terminal file before which to purge relay log files. Sample format: "mysql-bin.000
006"
  -h, --help              help for purge-relay
     --sub-dir string    specify relay sub directory for --filename. If not specified, the latest one will be used. Sam
ple format: "2ae76434-f79f-11e8-bde2-0242ac130008.000001"

Global Flags:
  -s, --source strings   MySQL Source ID
```

#### Command usage example

{{< copyable "" >}}

```bash
purge-relay -s "mysql-replica-01" --filename "mysql-bin.000003"
```

#### Flags description

- `-s`: (Required) Specifies the MySQL source for which to perform a clean operation
- `--filename`: (Required) Specifies the name of the terminal file before which to purge relay log files. For example, if the value is `mysql-bin.000100`, the clean operation stops at `mysql-bin.000099`.
- `--sub-dir`: (Optional) Specifies the relay log sub-directory corresponding to `--filename`. If not specified, the latest one is used.

#### Returned results

{{< copyable "" >}}

```bash
purge-relay -s "mysql-replica-01" --filename "mysql-bin.000003"
```

```
[warn] no --sub-dir specified for --filename; the latest one will be used
{
    "result": true,
    "msg": "",
    sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "
        }
    ]
}
```

### Preset skip operation

You can use `sql-skip` to preset a skip operation to be executed when the position or the SQL statement of the binlog event matches with the specified `binlog-pos` or `sql-pattern`. For descriptions of related parameters and results, refer to [`sql-skip`](skip-or-replace-abnormal-sql-statements.md#sql-skip).

### Preset replace operation

You can use `sql-replace` to preset a replace operation to be executed when the position or the SQL statement of the binlog event matches with the specified `binlog-pos` or `sql-pattern`. For descriptions of related parameters and results, refer to [`sql-replace`](skip-or-replace-abnormal-sql-statements.md#sql-replace).

## dmctl command mode

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

## Deprecated or unrecommended commands

The following commands are either deprecated or only used for debugging purposes. They might be completely removed or their semantics might be changed in future versions. **Strongly Not Recommended**.

- `migrate-relay`
- `sql-inject`
- `update-master-config`
- `update-relay`
