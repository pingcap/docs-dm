---
title: 处理出错的 SQL 语句
summary: 了解 TiDB Data Migration 如何处理出错的 SQL 语句。
aliases: ['/docs-cn/tidb-data-migration/dev/handle-error-sql-statements/']
---

# 处理出错的 SQL 语句

本文介绍了如何使用 DM 来处理出错的 SQL 语句。

目前，TiDB 并不完全兼容所有的 MySQL 语法（详见 [TiDB 已支持的 DDL 语句](https://pingcap.com/docs-cn/dev/reference/mysql-compatibility/#ddl)）。当使用 DM 从 MySQL 同步数据到 TiDB 时，如果 TiDB 不支持对应的 SQL 语句，可能会造成错误并中断同步任务。在这种情况下，DM 提供 `handle-error` 命令来恢复同步。

## 使用限制

如果业务不能接受下游 TiDB 跳过异常的 DDL 语句，也不接受使用其他 DDL 语句作为替代，则不适合使用此方式进行处理。

- 比如：`DROP PRIMARY KEY`，这种情况下，只能在下游重建一个（DDL 执行完后的）新表结构对应的表，并将原表的全部数据重新导入该新表。

### 支持场景

同步过程中，上游执行了 TiDB 不支持的 DDL 语句并同步到了 DM，造成同步任务中断。

- 如果业务能接受下游 TiDB 不执行该 DDL 语句，则使用 `handle-error <task-name> skip` 跳过对该 DDL 语句的同步以恢复同步任务。
- 如果业务能接受下游 TiDB 执行其他 DDL 语句来作为替代，则使用 `handle-error <task-name> replace` 替代该 DDL 的同步以恢复同步任务。

### 命令介绍

使用 dmctl 手动处理出错的 SQL 语句时，主要使用的命令包括 `query-status`、`handle-error`。

#### query-status

`query-status` 命令用于查询当前 MySQL 实例内子任务及 relay 单元等的状态和错误信息，详见[查询状态](query-status.md)。

#### handle-error

`handle-error` 命令用于处理错误的 sql 语句。

##### 命令用法

```bash
» handle-error -h
```

```
Usage:
  dmctl handle-error <task-name | task-file> [-s source ...] [-b binlog-pos] <skip/replace/revert> [replace-sql1;replace-sql2;] [flags]

Flags:
  -b, --binlog-pos string   position used to match binlog event if matched the handler-error operation will be applied. The format like "mysql-bin|000001.000003:3270"
  -h, --help                help for handle-error

Global Flags:
  -s, --source strings   MySQL Source ID
```

##### 参数解释

+ `task-name`：
    - 非 flag 参数，string，必选；
    - 指定预设的操作将生效的任务。

+ `source`：
    - flag 参数，string，`--source`；
    - `source` 指定预设操作将生效的 MySQL 实例。

+ `skip`：跳过该错误

+ `replace`：替代错误的 DDL 语句

+ `revert`：重置该错误先前的 skip/replace 操作

+ `binlog-pos`：
    - flag 参数，string，`--binlog-pos`；
    - 在指定时表示操作将在 `binlog-pos` 与 binlog event 的 position 匹配时生效，格式为 `binlog-filename:binlog-pos`，如 `mysql-bin|000001.000003:3270`。
    - 在同步执行出错后，binlog position 可直接从 `query-status` 返回的 `startLocation` 中的 `position` 获得。

### 使用示例

#### 同步中断执行跳过操作

##### 应用场景

假设现在存在如下四个上游表需要合并同步到下游的同一个表 ``` `shard_db`.`shard_table` ```，任务模式为悲观协调模式：

- MySQL 实例 1 内有 `shard_db_1` 逻辑库，包括 `shard_table_1` 和 `shard_table_2` 两个表。
- MySQL 实例 2 内有 `shard_db_2` 逻辑库，包括 `shard_table_1` 和 `shard_table_2` 两个表。

初始时表结构为：

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE shard_db.shard_table;
```

```
+-------+-----------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                              |
+-------+-----------------------------------------------------------------------------------------------------------+
| tb    | CREATE TABLE `shard_table` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin |
+-------+-----------------------------------------------------------------------------------------------------------+
```

此时，在上游所有分表上都执行以下 DDL 操作修改表字符集

{{< copyable "sql" >}}

```sql
ALTER TABLE `shard_db_*`.`shard_table_*` CHARACTER SET LATIN1 COLLATE LATIN1_DANISH_CI
```

则会由于 TiDB 不支持该 DDL 语句而导致 DM 同步任务中断，使用 `query-status` 命令查看错误信息：

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-01",
                "worker": "worker1",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Sync",
                    "result": {
                        "isCanceled": false,
                        "errors": [
                            {
                                "ErrCode": 44006,
                                "ErrClass": "schema-tracker",
                                "ErrScope": "internal",
                                "ErrLevel": "high",
                                "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1420), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4]: cannot track DDL: ALTER TABLE `shard_db_1`.`shard_table_1` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI",
                                "RawCause": "[ddl:8200]Unsupported modify charset from latin1 to utf8",
                                "Workaround": ""
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "sync": {
                        "totalEvents": "2",
                        "totalTps": "0",
                        "recentTps": "0",
                        "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1848)",
                        "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-8",
                        "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1420)",
                        "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                        "blockingDDLs": [
                        ],
                        "unresolvedGroups": [
                            {
                                "target": "`shard_db`.`shard_table`",
                                "DDLs": [
                                    "ALTER TABLE `shard_db`.`shard_table` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI"
                                ],
                                "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1420), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                                "synced": [
                                    "`shard_db_1`.`shard_table_1`"
                                ],
                                "unsynced": [
                                    "`shard_db_1`.`shard_table_2`"
                                ]
                            }
                        ],
                        "synced": false,
                        "binlogType": "remote"
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-02",
                "worker": "worker2",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Sync",
                    "result": {
                        "isCanceled": false,
                        "errors": [
                            {
                                "ErrCode": 44006,
                                "ErrClass": "schema-tracker",
                                "ErrScope": "internal",
                                "ErrLevel": "high",
                                "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1420), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-7]: cannot track DDL: ALTER TABLE `shard_db_2`.`shard_table_1` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI",
                                "RawCause": "[ddl:8200]Unsupported modify charset from latin1 to utf8",
                                "Workaround": ""
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "sync": {
                        "totalEvents": "2",
                        "totalTps": "0",
                        "recentTps": "0",
                        "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1848)",
                        "masterBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-8",
                        "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1420)",
                        "syncerBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6",
                        "blockingDDLs": [
                        ],
                        "unresolvedGroups": [
                            {
                                "target": "`shard_db`.`shard_table`",
                                "DDLs": [
                                    "ALTER TABLE `shard_db`.`shard_table` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI"
                                ],
                                "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1420), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6",
                                "synced": [
                                    "`shard_db_2`.`shard_table_1`"
                                ],
                                "unsynced": [
                                    "`shard_db_2`.`shard_table_2`"
                                ]
                            }
                        ],
                        "synced": false,
                        "binlogType": "remote"
                    }
                }
            ]
        }
    ]
}
```

可以看到 MySQL 实例1的 `shard_db_1`.`shard_table_1`表和 MySQL 实例2的 `shard_db_2`.`shard_table_1` 表报错。假设业务上可以接受下游 TiDB 不执行此 DDL 语句（即继续保持原有的表结构），则可以通过使用 `handle-error <task-name> skip` 命令跳过该 DDL 语句以恢复同步任务。操作步骤如下：

1. 使用 `handle-error <task-name> skip` 跳过 MySQL 实例1和实例2当前错误的 DDL 语句。

    {{< copyable "" >}}

    ```bash
    » hanlde-error test skip
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
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

2. 使用 `query-status <task-name>` 查看任务状态

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Paused",
                        "unit": "Sync",
                        "result": {
                            "isCanceled": false,
                            "errors": [
                                {
                                    "ErrCode": 44006,
                                    "ErrClass": "schema-tracker",
                                    "ErrScope": "internal",
                                    "ErrLevel": "high",
                                    "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1848), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4]: cannot track DDL: ALTER TABLE `shard_db_1`.`shard_table_2` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI",
                                    "RawCause": "[ddl:8200]Unsupported modify charset from latin1 to utf8",
                                    "Workaround": ""
                                }
                            ],
                            "detail": null
                        },
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "2",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1848)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-8",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1420)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                                {
                                    "target": "`shard_db`.`shard_table`",
                                    "DDLs": [
                                        "ALTER TABLE `shard_db`.`shard_table` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI"
                                    ],
                                    "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                                    "synced": [
                                        "`shard_db_1`.`shard_table_2`"
                                    ],
                                    "unsynced": [
                                        "`shard_db_1`.`shard_table_1`"
                                    ]
                                }
                            ],
                            "synced": false,
                            "binlogType": "remote"
                        }
                    }
                ]
            },
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-02",
                    "worker": "worker2",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Paused",
                        "unit": "Sync",
                        "result": {
                            "isCanceled": false,
                            "errors": [
                                {
                                    "ErrCode": 44006,
                                    "ErrClass": "schema-tracker",
                                    "ErrScope": "internal",
                                    "ErrLevel": "high",
                                    "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-7], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1848), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-8]: cannot track DDL: ALTER TABLE `shard_db_2`.`shard_table_2` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI",
                                    "RawCause": "[ddl:8200]Unsupported modify charset from latin1 to utf8",
                                    "Workaround": ""
                                }
                            ],
                            "detail": null
                        },
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "2",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1848)",
                            "masterBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-8",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1420)",
                            "syncerBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                                {
                                    "target": "`shard_db`.`tb`",
                                    "DDLs": [
                                        "ALTER TABLE `shard_db`.`tb` CHARACTER SET UTF8 COLLATE UTF8_UNICODE_CI"
                                    ],
                                    "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1634), gtid-set: 1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-7",
                                    "synced": [
                                        "`shard_db_2`.`shard_table_2`"
                                    ],
                                    "unsynced": [
                                        "`shard_db_2`.`shard_table_1`"
                                    ]
                                }
                            ],
                            "synced": false,
                            "binlogType": "remote"
                        }
                    }
                ]
            }
        ]
    }
    ```

    可以看到 MySQL 实例1的 `shard_db_1`.`shard_table_2` 表和 MySQL 实例2的 `shard_db_2`.`shard_table_2` 表报错，MySQL 实例1的 `shard_db_1`.`shard_table_1`表和 MySQL 实例2的 `shard_db_2`.`shard_table_1` 表的 DDL 语句已经被跳过。

3. 继续使用 `handle-error <task-name> skip` 跳过 MySQL 实例1和实例2当前错误的 DDL 语句。

    {{< copyable "" >}}

    ```bash
    » hanlde-error test skip
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
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

4. 使用 `query-status <task-name>` 查看任务状态

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "4",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": true,
                            "binlogType": "remote"
                        }
                    }
                ]
            },
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-02",
                    "worker": "worker2",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "4",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2268)",
                            "masterBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2268)",
                            "syncerBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": try,
                            "binlogType": "remote"
                        }
                    }
                ]
            }
        ]
    }
    ```

    可以看到任务运行正常，无错误信息。四条 DDL 全部被跳过。

#### 同步中断执行替代操作

##### 应用场景

假设现在存在如下四个上游表需要合并同步到下游的同一个表 ``` `shard_db`.`shard_table` ```，任务模式为悲观协调模式：

- MySQL 实例 1 内有 `shard_db_1` 逻辑库，包括 `shard_table_1` 和 `shard_table_2` 两个表。
- MySQL 实例 2 内有 `shard_db_2` 逻辑库，包括 `shard_table_1` 和 `shard_table_2` 两个表。

初始时表结构为：

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE shard_db.shard_table;
```

```
+-------+-----------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                              |
+-------+-----------------------------------------------------------------------------------------------------------+
| tb    | CREATE TABLE `shard_table` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin |
+-------+-----------------------------------------------------------------------------------------------------------+
```

此时，在上游所有分表上都执行以下 DDL 操作增加新列，并添加 UNIQUE 约束

{{< copyable "sql" >}}

```sql
ALTER TABLE `shard_db_*`.`shard_table_*` ADD COLUMN new_col INT UNIQUE
```

则会由于 TiDB 不支持该 DDL 语句而导致 DM 同步任务中断，使用 `query-status` 命令查看错误信息：

```
» query-status test
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-01",
                "worker": "worker1",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Sync",
                    "result": {
                        "isCanceled": false,
                        "errors": [
                            {
                                "ErrCode": 44006,
                                "ErrClass": "schema-tracker",
                                "ErrScope": "internal",
                                "ErrLevel": "high",
                                "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5]: cannot track DDL: ALTER TABLE `shard_db_1`.`shard_table_1` ADD COLUMN `new_col` INT UNIQUE KEY",
                                "RawCause": "[ddl:8200]unsupported add column 'new_col' constraint UNIQUE KEY when altering 'shard_db_1.shard_table_1'",
                                "Workaround": ""
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "sync": {
                        "totalEvents": "0",
                        "totalTps": "0",
                        "recentTps": "0",
                        "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1485)",
                        "masterBinlogGtid": "6acc73aa-dd57-11ea-8d82-00155de45f57:1-7",
                        "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1095)",
                        "syncerBinlogGtid": "6acc73aa-dd57-11ea-8d82-00155de45f57:1-5",
                        "blockingDDLs": [
                        ],
                        "unresolvedGroups": [
                            {
                                "target": "`shard_db`.`shard_table`",
                                "DDLs": [
                                    "ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT UNIQUE KEY"
                                ],
                                "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5",
                                "synced": [
                                    "`shard_db_1`.`shard_table_1`"
                                ],
                                "unsynced": [
                                    "`shard_db_1`.`shard_table_2`"
                                ]
                            }
                        ],
                        "synced": false,
                        "binlogType": "remote"
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-02",
                "worker": "worker2",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Sync",
                    "result": {
                        "isCanceled": false,
                        "errors": [
                            {
                                "ErrCode": 44006,
                                "ErrClass": "schema-tracker",
                                "ErrScope": "internal",
                                "ErrLevel": "high",
                                "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-6]: cannot track DDL: ALTER TABLE `shard_db_2`.`shard_table_1` ADD COLUMN `new_col` INT UNIQUE KEY",
                                "RawCause": "[ddl:8200]unsupported add column 'new_col' constraint UNIQUE KEY when altering 'shard_db_2.shard_table_1'",
                                "Workaround": ""
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "sync": {
                        "totalEvents": "0",
                        "totalTps": "0",
                        "recentTps": "0",
                        "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1485)",
                        "masterBinlogGtid": "6dcf281d-dd57-11ea-8f7c-00155de45f57:1-7",
                        "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1095)",
                        "syncerBinlogGtid": "6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5",
                        "blockingDDLs": [
                        ],
                        "unresolvedGroups": [
                            {
                                "target": "`shard_db`.`shard_table`",
                                "DDLs": [
                                    "ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT UNIQUE KEY"
                                ],
                                "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5",
                                "synced": [
                                    "`shard_db_2`.`shard_table_1`"
                                ],
                                "unsynced": [
                                    "`shard_db_2`.`shard_table_2`"
                                ]
                            }
                        ],
                        "synced": false,
                        "binlogType": "remote"
                    }
                }
            ]
        }
    ]
}
```

可以看到 MySQL 实例1的 `shard_db_1`.`shard_table_1`表和 MySQL 实例2的 `shard_db_2`.`shard_table_1` 表报错。我们将该 DDL 替换成两条等价的 DDL 。操作步骤如下：

1. 使用如下命令分别替换 MySQL 实例1和实例2中错误的 DDL 语句。

    {{< copyable "" >}}

    ```bash
    » handle-error test -s mysql-replica-01 replace "ALTER TABLE `shard_db_1`.`shard_table_1` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_1`.`shard_table_1` ADD UNIQUE(`new_col`)";
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

    {{< copyable "" >}}

    ```bash
    » handle-error test -s mysql-replica-02 replace "ALTER TABLE `shard_db_2`.`shard_table_1` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_2`.`shard_table_1` ADD UNIQUE(`new_col`)";
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

2. 使用 `query-status <task-name>` 查看任务状态

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Paused",
                        "unit": "Sync",
                        "result": {
                            "isCanceled": false,
                            "errors": [
                                {
                                    "ErrCode": 36008,
                                    "ErrClass": "sync-unit",
                                    "ErrScope": "internal",
                                    "ErrLevel": "high",
                                    "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1485), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5]: detect inconsistent DDL sequence from source [first-location: position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5 ddls: [ALTER TABLE `shard_db`.`tb` ADD COLUMN `new_col` INT UNIQUE KEY] source: `shard_db_1`.`shard_table_2`], right DDL sequence should be [first-location: position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5 ddls: [ALTER TABLE `handle_error`.`tb` ADD COLUMN `new_col` INT] source: `shard_db_1`.`shard_table_1` first-location: position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5, suffix: 1 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD UNIQUE(`new_col`)] source: `shard_db`.`shard_table_1` first-location: position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT UNIQUE KEY] source: `shard_db`.`shard_table_2`]",
                                    "RawCause": "",
                                    "Workaround": "Please use `show-ddl-locks` command for more details."
                                }
                            ],
                            "detail": null
                        },
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "0",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1485)",
                            "masterBinlogGtid": "6acc73aa-dd57-11ea-8d82-00155de45f57:1-7",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1095)",
                            "syncerBinlogGtid": "6acc73aa-dd57-11ea-8d82-00155de45f57:1-5",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                                {
                                    "target": "`shard_db`.`shard_table`",
                                    "DDLs": [
                                        "ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT"
                                    ],
                                    "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6acc73aa-dd57-11ea-8d82-00155de45f57:1-5",
                                    "synced": [
                                        "`shard_db_1`.`shard_table_1`"
                                    ],
                                    "unsynced": [
                                        "`shard_db_1`.`shard_table_2`"
                                    ]
                                }
                            ],
                            "synced": false,
                            "binlogType": "remote"
                        }
                    }
                ]
            },
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-02",
                    "worker": "worker2",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Paused",
                        "unit": "Sync",
                        "result": {
                            "isCanceled": false,
                            "errors": [
                                {
                                    "ErrCode": 36008,
                                    "ErrClass": "sync-unit",
                                    "ErrScope": "internal",
                                    "ErrLevel": "high",
                                    "Message": "startLocation: [position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-6], endLocation: [position: (DESKTOP-T561TSO-bin.000001, 1485), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-7]: detect inconsistent DDL sequence from source [first-location: position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-6 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT UNIQUE KEY] source: `shard_db_2`.`shard_table_2`], right DDL sequence should be [first-location: position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT] source: `shard_db_2`.`shard_table_1` first-location: position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5, suffix: 1 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD UNIQUE(`new_col`)] source: `shard_db_2`.`shard_table_1` first-location: position: (DESKTOP-T561TSO-bin.000001, 1290), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-6 ddls: [ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT UNIQUE KEY] source: `shard_db_2`.`shard_table_2`]",
                                    "RawCause": "",
                                    "Workaround": "Please use `show-ddl-locks` command for more details."
                                }
                            ],
                            "detail": null
                        },
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "0",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 1485)",
                            "masterBinlogGtid": "6dcf281d-dd57-11ea-8f7c-00155de45f57:1-7",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 1095)",
                            "syncerBinlogGtid": "6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                                {
                                    "target": "`shard_db`.`shard_table`",
                                    "DDLs": [
                                        "ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `new_col` INT"
                                    ],
                                    "firstLocation": "position: (DESKTOP-T561TSO-bin.000001, 1095), gtid-set: 6dcf281d-dd57-11ea-8f7c-00155de45f57:1-5",
                                    "synced": [
                                        "`shard_db2`.`shard_table_1`"
                                    ],
                                    "unsynced": [
                                        "`shard_db2`.`shard_table_2`"
                                    ]
                                }
                            ],
                            "synced": false,
                            "binlogType": "remote"
                        }
                    }
                ]
            }
        ]
    }
    ```

    可以看到 MySQL 实例1的 `shard_db_1`.`shard_table_2` 表和 MySQL 实例2的 `shard_db_2`.`shard_table_2` 表报错，MySQL 实例1的 `shard_db_1`.`shard_table_1`表和 MySQL 实例2的 `shard_db_2`.`shard_table_1` 表的 DDL 语句已经被替换成功。

3. 使用如下命令继续分别替换 MySQL 实例1和实例2中错误的 DDL 语句。

    {{< copyable "" >}}

    ```bash
    » handle-error test -s mysql-replica-01 replace "ALTER TABLE `shard_db_1`.`shard_table_2` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_1`.`shard_table_2` ADD UNIQUE(`new_col`)";
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

    {{< copyable "" >}}

    ```bash
    » handle-error test -s mysql-replica-02 replace "ALTER TABLE `shard_db_2`.`shard_table_2` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_2`.`shard_table_2` ADD UNIQUE(`new_col`)";
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

4. 使用 `query-status <task-name>` 查看任务状态

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "4",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": true,
                            "binlogType": "remote"
                        }
                    }
                ]
            },
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-02",
                    "worker": "worker2",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "totalEvents": "4",
                            "totalTps": "0",
                            "recentTps": "0",
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2268)",
                            "masterBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2268)",
                            "syncerBinlogGtid": "1d66ce48-dd4a-11ea-8ef7-00155de45f57:1-6",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": try,
                            "binlogType": "remote"
                        }
                    }
                ]
            }
        ]
    }
    ```

    可以看到任务运行正常，无错误信息。四条 DDL 全部被替换。
