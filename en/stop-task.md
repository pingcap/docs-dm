---
title: Stop a Data Replication Task
summary: Learn how to stop a data replication task.
---

# Stop a Data Replication Task

You can use the `stop-task` command to stop a data replication task. For differences between `stop-task` and `pause-task`, refer to [Pause a Data Replication Task](pause-task.md).

{{< copyable "" >}}

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

## Usage example

{{< copyable "" >}}

```bash
stop-task [-s "mysql-replica-01"]  task-name
```

## Flags description

- `-s`: (Optional) Specifies the MySQL source where the subtasks of the replication task (that you want to stop) run. If it is set, only subtasks on the specified MySQL source are stopped.
- `task-name`: (Required) Specifies the task name.

## Returned results

{{< copyable "" >}}

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
