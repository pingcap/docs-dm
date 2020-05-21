---
title: 查询数据同步任务信息
summary: 了解 TiDB Data Migration 如何查询数据同步任务信息。
category: reference
---

# 查询数据同步任务信息

本文介绍了如何使用 [dmctl](overview.md#dmctl) 组件来查询任务状态和任务错误。

## 查询数据同步任务状态

`query-status` 命令用于查询数据同步任务状态。有关查询结果及子任务状态，详见[查询状态](query-status.md)。

{{< copyable "" >}}

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

### 命令用法示例

{{< copyable "" >}}

```bash
query-status
```

### 参数解释

- `-s`：
    - 可选
    - 查询在指定的一个 MySQL 源上运行的数据同步任务的子任务
- `task-name`：
    - 可选
    - 指定任务名称
    - 如果未设置，则返回全部数据同步任务的查询结果

### 返回结果示例

有关查询结果中各参数的意义，详见[查询状态结果](query-status.md#查询结果)。

## 查询数据同步任务错误

`query-error` 可用于查询数据同步任务与 relay 处理单元的错误信息。相比于 `query-status`，`query-error` 一般不用于获取除错误信息之外的其他信息。有关查询结果及子任务状态，详见[查询错误](query-error.md)。

`query-error` 常用于获取 `sql-skip`/`sql-replace` 所需的 binlog position 信息，有关 `sql-skip` 和 `sql-replace` 的介绍，请参考 [跳过或替代执行异常的 SQL 语句](skip-or-replace-abnormal-sql-statements.md#query-error)。

{{< copyable "" >}}

```bash
help query-error
```

```
query task error

Usage:
 dmctl query-error [-s source ...] [task-name] [flags]

Flags:
 -h, --help   help for query-error

Global Flags:
 -s, --source strings   MySQL Source ID
```

### 命令用法示例

{{< copyable "" >}}

```bash
query-error
```

### 参数解释

- `-s`：
    - 可选
    - 查询在指定的一个 MySQL 源上运行的数据同步任务的子任务
- `task-name`：
    - 可选
    - 指定任务名称
    - 如果未设置，则返回全部数据同步任务错误的查询结果

### 返回结果示例

有关查询错误结果中各参数的意义，详见[查询错误结果](query-error.md#查询结果)。
