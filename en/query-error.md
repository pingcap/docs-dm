---
title: Query Error
summary: Learn how to query error that occurs in the running of a data replication task and subtasks.
---

# Query Error

You can use `query-error` to query error that occurs in the running of a data replication task and subtasks.

## `query-error`

{{< copyable "" >}}

```bash
» query-error
```

```
{
    "result": true,                              # Whether query-error is executed successfully.
    "msg": "",                                   # The description of why query-error fails.
    "sources": [                                 # The source information list.
        {
            "result": true,                      # Whether query-error is executed successfully on this source.
            "msg": "",                           # The description of why query-error fails on this source.
            "SourceError": {                     # The source information.
                "source": "mysql-replica-01",
                "worker": "worker1",
                "SourceError": "",
                "RelayError": null
            },
            "subTaskError": [                    # The error messages of all subtasks running on the source.
                {
                    "name": "test",
                    "stage": "Running",
                    "unit": "Sync",
                    "sync": {
                        "errors": [
                        ]
                    }
                },
                {
                    "name": "test2",
                    "stage": "Paused",
                    "unit": "Sync",
                    "sync": {
                        "errors": [
                        ]
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
            "SourceError": {
                "source": "mysql-replica-02",
                "worker": "worker2",
                "SourceError": "",
                "RelayError": null
            },
            "subTaskError": [
                {
                    "name": "test",
                    "stage": "Running",
                    "unit": "Sync",
                    "sync": {
                        "errors": [
                        ]
                    }
                },
                {
                    "name": "test2",
                    "stage": "Paused",
                    "unit": "Sync",
                    "sync": {
                        "errors": [
                        ]
                    }
                }
            ]
        }
    ]
}
```

## Query subtask errors

{{< copyable "" >}}

```bash
» query-status test
```

```
{
    "result": true,                              # Whether query-error is executed successfully.
    "msg": "",                                   # The description of why query-error fails.
    "sources": [                                 # The source information list.
        {
            "result": true,                      # Whether query-error is executed successfully on this source.
            "msg": "",                           # The description of why query-error fails on this source.
            "SourceError": {                     # The source information.
                "source": "mysql-replica-01",
                "worker": "worker1",
                "SourceError": "",
                "RelayError": null
            },
            "subTaskError": [                    # The error information of subtasks running on this source.
                {
                    "name": "test",              # The task name.
                    "stage": "Paused",           # The current task status.
                    "unit": "Sync",              # The unit currently processing the task.
                    "sync": {                    # The error message of the binlog replication unit (Sync).
                        "errors": [              # The list of error messages of the current processing unit.
                            {
                                // The description of the error message.
                                "msg": "exec sqls[[USE `db1`; ALTER TABLE `db1`.`tbl1` CHANGE COLUMN `c2` `c2` decimal(10,3);]] failed, err:Error 1105: unsupported modify column length 10 is less than origin 11",
                                // The position of the binlog event where the error occurs.
                                "failedBinlogPosition": "mysql-bin|000001.000003:34642",
                                // The SQL statement with error.
                                "errorSQL": "[USE `db1`; ALTER TABLE `db1`.`tbl1` CHANGE COLUMN `c2` `c2` decimal(10,3);]"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
```
