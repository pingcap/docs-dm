---
title: 使用 SQL 表达式过滤某些行变更
---

# 使用 SQL 表达式过滤某些行变更

## 概述

在进行数据同步的过程中，DM 提供了 [Binlog Event Filter](key-features.md#binlog-event-filter)可以过滤某些类型的 binlog event，例如不向下游同步 DELETE 以达到归档、审计等目的。但是 Binlog Event Filter 无法以更细粒度判断某一行的 DELETE 是否要被过滤。

为了解决上面的问题，DM 在 v2.0.5 版本可以使用 SQL 表达式过滤某些行变更。DM 所要求的 ROW 模式的 binlog 会在 binlog event 中带有所有列的值。基于这些值，用户可以配置 SQL 表达式，如果该表达式对于某条行变更的计算结果是 `TRUE`，DM 就不会向下游同步该条行变更。

## 配置示例

表达式过滤在 task 配置文件里面与 [Binlog Event Filter](key-features.md#binlog-event-filter) 类似，详见下面配置样例。完整的配置及意义，可以参考 [DM 完整配置文件示例](task-configuration-file-full.md#完整配置文件示例)：

```yml
name: test
task-mode: all

target-database:
  host: "127.0.0.1"
  port: 4000
  user: "root"
  password: ""

mysql-instances:
  - source-id: "mysql-replica-01"
    expression-filters: ["even_c"]

expression-filter:
  even_c:
    schema: "expr_filter"
    table: "tbl"
    insert-value-expr: "c % 2 = 0"
```

上面的配置中配置了 `even_c` 规则，并让 source ID 为 `mysql-replica-01` 的上游引用了该规则。`even_c` 规则的含义是：

对于 `expr_filter` 库下的 `tbl` 表，当插入偶数的 `c` (`c % 2 = 0`) 时，不将这条插入同步到下游。

## 配置规则

- schema: 要匹配的上游数据库库名，不支持通配符匹配或正则匹配
- table: 要匹配的上游表名，不支持通配符匹配或正则匹配
- insert-value-expr: 可以配置一个表达式，对 INSERT 类型的 binlog event (WRITE_ROWS_EVENT) 带有的值生效。不能与 update-old-value-expr、update-new-value-expr、delete-value-expr 出现在一个配置项中。
- update-old-value-expr: 可以配置一个表达式，对 UPDATE 类型的 binlog event (UPDATE_ROWS_EVENT) 更新对应的旧值生效。不能与 insert-value-expr、delete-value-expr 出现在一个配置项中。
- update-new-value-expr: 可以配置一个表达式，对 UPDATE 类型的 binlog event (UPDATE_ROWS_EVENT) 更新对应的新值生效。不能与 insert-value-expr、delete-value-expr 出现在一个配置项中。
- delete-value-expr: 可以配置一个表达式，对 DELETE 类型的 binlog event (DELETE_ROWS_EVENT) 带有的值生效。不能与 insert-value-expr、update-old-value-expr、update-new-value-expr 出现在一个配置项中。

> **注意：**
>
> update-old-value-expr 可以与 update-new-value-expr 同时配置。
> - 当二者同时配置时，会将更新旧值满足 update-old-value-expr **且**更新新值满足 update-new-value-expr 的行变动过滤掉。
> - 当只配置一者时，配置的这条表达式会决定是否过滤**整个行变更**，即对旧值的删除和新值的插入会作为一个整体被过滤掉。

SQL 表达式可以涉及一列、多列、使用 TiDB 支持的 SQL 函数。例如 `c % 2 = 0`，`a*a + b*b = c*c`、`ts > NOW()`

TIMESTAMP 类型的默认时区是 UTC。可以使用 `c_timestamp = '2021-01-01 12:34:56.5678+08:00'` 的方式显式指定时区。

配置项 `expression-filter` 下可以配置多种过滤规则，上游数据源在 `expression-filters` 中引用这些规则使其生效。当有多条规则生效时，匹配到**任意一条**规则即会导致某个行变更被过滤。

> **注意：**
>
> 为某张表设置过多的表达式过滤会增加 DM 的计算开销，可能导致同步变慢。
