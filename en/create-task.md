---
title: Create a Data Replication Task
summary: Learn how to create a data replication task in TiDB Data Migration.
category: reference
---

# Create a Data Replication Task

You can use the `start-task` command to create a data replication task. When the data replication task is started, DM [prechecks privileges and configurations](precheck.md).

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

## Usage example

{{< copyable "" >}}

```bash
start-task [ -s "mysql-replica-01"] ./task.yaml
```

## Flags description

- `-s`: (Optional) Specifies the MySQL source to execute `task.yaml`. If it is set, the command only starts the subtasks of the specified task on the MySQL source.
- `config-file`: (Required) Specifies the file path of `task.yaml`.

## Returned results

{{< copyable "" >}}

```bash
start-task task.yaml
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
            "worker": "worker1"
        }
    ]
}
```
