---
title: Pause a Data Replication Task
summary: Learn how to pause a data replication task in TiDB Data Migration.
category: reference
---

# Pause a Data Replication Task

You can use the `pause-task` command to pause a data replication task.

`pause-task` differs from `stop-task` in that:

- `pause-task` only pauses a replication task. You can query the status information (retained in the memory) of the task using `query-status`. `stop-task` terminates a replication task and removes all information related to this task from the memory. This means you cannot use `query-status` to query the status information. `dm_meta` like "checkpoint" and data that have been replicated to the downstream are not removed.
- If `pause-task` is executed to pause the replication task, you cannot start a new task with the same name, neither can you get the relay log of the paused task removed, since this task does exist. If `stop-task` is executed to stop a task, you can start a new task with the same name, and you can get the relay log of the stopped task removed, since this task no longer exists.
- `pause-task` is usually used to pause a task for troubleshooting, while `stop-task` is to permanently remove a replication task, or to co-work with `start-task` to update the configuration information.

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

## Usage example

{{< copyable "" >}}

```bash
pause-task [-s "mysql-replica-01"] task-name
```

## Flags description

- `-s`: (Optional) Specifies the MySQL source where you want to pause the subtasks of the replication task. If it is set, this command pauses only the subtasks on the specified MySQL source.
- `task-name`: (Required) Specifies the task name.

## Returned results

{{< copyable "" >}}

```bash
pause-task test
```

```
{
    "op": "Pause",
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
