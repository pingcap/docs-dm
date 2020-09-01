---
title: Merge and Replicate Data from Sharded Tables
summary: Learn how DM merges and replicates data from sharded tables.
---

# Merge and Replicate Data from Sharded Tables

This document introduces the sharding support feature provided by Data Migration (DM). This feature allows you to merge and replicate the data of tables with the same or different table schemas in the upstream MySQL or MariaDB instances into one same table in the downstream TiDB. It supports not only replicating the upstream DML statements, but also coordinating to replicate the table schema change using DDL statements in multiple upstream sharded tables.

## Overview

DM supports merging and replicating the data of multiple upstream sharded tables into one table in TiDB. During the replication, the DDL of each sharded table, and the DML before and after the DDL need to be coordinated. For the usage scenarios, DM supports two different modes: pessimistic mode and optimistic mode.

> **Note:**
>
> - To merge and replicate data from sharded tables,  you must set `shard-mode` in the task configuration file. 
> - DM uses the pessimistic mode by default for the merge of the sharding support feature. (If there is no special description in the document, use the pessimistic mode by default.)
> - It is not recommended to use this mode if you do not understand the principles and restrictions of the optimistic mode. Otherwise, it may cause serious consequences such as replication interruption and even data inconsistency.

### The pessimistic mode

When an upstream sharded table executes a DDL statement, the replication of this sharded table will be suspended. After all other sharded tables execute the same DDL, the DDL will be executed in the downstream and the data replication task will restart. The advantage of this mode is that it can ensure that the data replicated to the downstream will not go wrong. For details, refer to [shard merge in pessimistic mode](feature-shard-merge-pessimistic.md).
 
### The optimistic mode

DM will automatically modify the DDL executed on a sharded table into a statement compatible with other sharded tables, and then replicate to the downstream. This will not block the DML replication of any sharded tables. The advantage of this mode is that it will not block data replication when processing DDL. However, improper use will cause replication interruption or even data inconsistency. For details, refer to [shard merge in optimistic mode](feature-shard-merge-optimistic.md).

### Contrast

| Pessimistic mode   | Optimistic mode   |
| :----------- | :----------- |
| Sharded tables that executes DDL suspend DML replication | Sharded tables that executes DDL continue DML replication |
| The DDL execution order and statements of each sharded table must be the same | Each sharded table only needs to keep the table schema compatible with each other  |
| The DDL is replicated to the downstream after the entire shard group is consistent | The DDL of each sharded table immediately affects the downstream |
| Wrong DDL operations can be intercepted after the detection | Wrong DDL operations will be replicated to the downstream, which may cause inconsistency between the upstream and downstream data before the detection  |