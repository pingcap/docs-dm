---
title: Optimize Configuration of DM
summary: Learn how to optimize the configuration of the data replication task to improve the performance of data replication.
category: reference
---

# Optimize Configuration of DM

This document introduces how to optimize the configuration of the data replication task to improve the performance of data replication.

## Full data export

`mydumpers` is a configuration item related to full data export. This section describes how to configure performance-related options.

### `rows`

Setting the `rows` option enables concurrently exporting data from a single table using multi-thread. The value of `rows` is the maximum number of rows contained in each exported chunk. After this option is enabled, DM selects a column as the split benchmark when the data of a MySQL single table is concurrently exported. This column can be one of the following collumns: the primary key column, the unique index column, and the normal index column (ordered from highest priority to lowest). Make sure this column is of integer type (for example, `INT`, `MEDIUMINT`, `BIGINT`). 

The value of `rows` can be set to 10000. You can change this value according to the total number of rows in the table and the performance of the database. In addition , need to set `threads` to control the number of concurrent threads, the default value is 4, can be adjusted appropriately.

### `chunk-filesize`

DM full backup will split the data of each table into multiple chunks according to the value of the `chunk-filesize` parameter, and each chunk is saved in a file with a size of about `chunk-filesize`. According to this parameter, the data is splitted into multiple files, so that the parallel processing logic of the DM Load processing unit can be used to increase the import speed. The default value of this parameter is 64 (the unit is MB). Under normal situation, it does not need to be set. Also can make appropriate adjustments according to the size of the entire size of data.

> **Note：**
>
> - The parameter value of `mydumpers` does not support updating after the replication task is created, so need to determine the value of each parameter before creating the task. If need to update, need to use dmctl stop the task to update the configuration file, and then re-create the task.

> - `mydumpers`.`threads` can be replaced with the configuration item `mydumper-thread` to simplify configuration.

> - If `rows` is set，DM will ingore the vaue of `chunk-filesize`. 

## Full import 

The configuration item related to full import is `loaders`. The following describes how to configure the parameters related to performance.

### `pool-size`

`pool-size` is the setting of the number of threads in the DM load phase. The default value is 16. Normally, there is no need to set it. Also can make appropriate adjustments based on the size of full size of data and the performance of the database.

> **Note：**
>
> - The parameter value of `loaders` does not support updating after the replication task is created, so need to define the value of each parameter before creating the task. If need to update, so need to use dmctl stop the task to update the configuration file, and then re-create the task.

> - `loaders`.`pool-size` can be replaced with the configuration item `loader-thread` to simplify configuration.

## Incremental replication

The configuration related to incremental replication is `syncers`. The following describes how to configure the parameters related to performance.

### `worker-count`

`worker-count` is the number of threads for concurrent replication DML in the DM sync phase. The default value is 16. If there is a high requirement for replication speed, the value the parameter can be adjusted appropriately.

### batch

`batch` is the number of DML included in each transaction when the data is replicated to the downstream database in the DM sync phase. The default value is 100. Normally, no adjustment is required.

> **Note：**
>
> - The parameter value of `syncers` does not support updating after the replication task is created, so need to define the value of each parameter before creating the task. If need to update, so need to use dmctl stop the task to update the configuration file, and then re-create the task.
> - `syncers`.`worker-count` can be replaced with the configuration item `syncer-thread` to simplify configuration.
> - The settings of `worker-count` and `batch` need to be adjusted according to the actual situation, for example: the network delay from the DM to the downstream database is high, can appropriately increase the `worker-count` and lower the `batch`.
