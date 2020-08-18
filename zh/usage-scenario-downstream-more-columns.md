---
title: 下游表结构存在更多列的迁移场景
summary: 了解如何在下游表结构比上游存在更多列的情况下，使用 DM 对表进行迁移。
---

# 下游表结构存在更多列的迁移场景

本文介绍如何在下游表结构比上游存在更多列的情况下，使用 DM 对表进行迁移。

> **注意：**
>
> - 下游相比于上游多的列必须指定默认值或允许为 `NULL`。
> - 对于已经通过 DM 在正常迁移的表，可直接在下游添加指定了默认值或允许为 `NULL` 的列而不会影响数据迁移。
> - 当前 DM 暂不支持全量导入表结构不一致的数据到 TiDB 中。

## 下游已有表存在更多列的数据迁移

假如在进行数据同步前，上游表结构为：

```sql
CREATE TABLE `t1` (
  `c1` int(11) NOT NULL,
  `c2` int(11) DEFAULT NULL,
  PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

下游表结构为：

```sql
CREATE TABLE `t1` (
  `c1` int(11) NOT NULL,
  `c2` int(11) DEFAULT NULL,
  `c3` varchar(256) DEFAULT 'default_c3',
  PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin
```

当使用 DM 迁移增量数据时，如果下游表结构中存在比上游表结构更多的列，则通过 `query-status` 可看到类似如下的错误：

```
"errors": [
    {
        "ErrCode": 36027,
        "ErrClass": "sync-unit",
        "ErrScope": "internal",
        "ErrLevel": "high",
        "Message": "startLocation: [position: (mysql-bin.000001, 17491), gtid-set: ], endLocation: [position: (mysql-bin.000001, 17535), gtid-set: ]: gen insert sqls failed, schema: db_single, table: t1: Column count doesn't match value count: 3 (columns) vs 2 (values)",
        "RawCause": "",
        "Workaround": ""
    }
]
```

DM 在遇到 binlog event 里的 DML 需要迁移时，如果 DM 内部没有维护对应于该表的表结构，则会尝试使用下游当前的表结构来解析 binlog event 并生成相应的 DML 语句。如果 binlog event 里数据的列数与下游表结构的列数不一致时，则会产生上述错误。

此时，我们可以使用 [`operate-schema`](manage-schema.md) 命令来为该表指定与 binlog event 匹配的表结构，具体操作为：

1. 准备与 binlog event 里数据对应的 `CREATE TABLE` 表结构语句并保存到文件，如将以下表结构保存到 `db_single.t1-schema.sql` 中。

    ```sql
    CREATE TABLE `t1` (
      `c1` int(11) NOT NULL,
      `c2` int(11) DEFAULT NULL,
      PRIMARY KEY (`c1`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1
    ```

2. 使用 [`operate-schema`](manage-schema.md) 命令设置表结构（此时 task 应该由于上述错误而处于 `Paused` 状态）。

    {{< copyable "" >}}
    
    ```bash
    operate-schema set -s mysql-replica-01 task_single -d db_single -t t1 db_single.t1-schema.sql
    ```    

3. 使用 [`resume-task`](resume-task.md) 命令恢复处于 `Paused` 状态的任务。

4. 使用 [`query-status`](query-status.md) 命令确认数据迁移任务是否运行正常。
