---
title: TiDB Data Migration 查询运行错误
summary: 深入了解 TiDB Data Migration 如何查询数据同步任务运行错误。
category: reference
---

# TiDB Data Migration 查询运行错误

本文介绍 DM（Data Migration）`query-error` 命令的查询错误与子任务错误。

## 查询结果

{{< copyable "" >}}

```bash
» query-error
```

```
{
    "result": true,                              # query-error 操作本身是否成功
    "msg": "",                                   # query-error 操作失败的说明信息
    "sources": [                                 # source 信息列表
        {
            "result": true,                      # 该 source 上 query-error 操作是否成功
            "msg": "",                           # 该 source 上 query-error 操作失败的说明信息
            "SourceError": {                     # 该 source 信息
                "source": "mysql-replica-01",
                "worker": "worker1",
                "SourceError": "",
                "RelayError": null
            },
            "subTaskError": [                    # 该 source 上运行的所有子任务的错误信息
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

## 查询子任务错误

{{< copyable "" >}}

```bash
» query-status test
```

```
{
    "result": true,                              # query-error 操作本身是否成功
    "msg": "",                                   # query-error 操作失败的说明信息
    "sources": [                                 # source 信息列表
        {
            "result": true,                      # 该 source 上 query-error 操作是否成功
            "msg": "",                           # 该 source 上 query-error 操作失败的说明信息
            "SourceError": {                     # 该 source 信息
                "source": "mysql-replica-01",
                "worker": "worker1",
                "SourceError": "",
                "RelayError": null
            },
            "subTaskError": [                    # 该 source 上运行子任务的错误信息
                {
                    "name": "test",              # 任务名
                    "stage": "Paused",           # 当前任务的状态
                    "unit": "Sync",              # 当前正在处理任务的处理单元
                    "sync": {                    # binlog 同步单元（sync）的错误信息
                        "errors": [              # 当前处理单元的错误信息列表
                            {
                                // 错误信息描述
                                "msg": "exec sqls[[USE `db1`; ALTER TABLE `db1`.`tbl1` CHANGE COLUMN `c2` `c2` decimal(10,3);]] failed, err:Error 1105: unsupported modify column length 10 is less than origin 11",
                                // 发生错误的 binlog event 的 position
                                "failedBinlogPosition": "mysql-bin|000001.000003:34642",
                                // 发生错误的 SQL 语句
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
