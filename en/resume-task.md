---
title: Resume a Data Replication Task
summary: Learn how to resume a data replication task.
---

# Resume a Data Replication Task

You can use the `resume-task` command to resume a data replication task in the `Paused` state. This is generally used in scenarios where you want to manually resume a data replication task after handling the error that get the task paused.

{{< copyable "" >}}

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

## Usage example

{{< copyable "" >}}

```bash
resume-task [-s "mysql-replica-01"] task-name
```

## Flags description

- `-s`: (Optional) Specifies the MySQL source where you want to resume the subtask of the replication task. If it is set, the command resumes only the subtasks on the specified MySQL source.
- `task-name`: (Required) Specifies the task name.

## Returned results

{{< copyable "" >}}

```bash
resume-task test
```

```
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
