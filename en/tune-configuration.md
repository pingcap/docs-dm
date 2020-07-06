---
title: Optimize Configuration of DM
summary: Learn how to optimize the configuration of DM to improve the performance of data replication
category: reference
---

# DM configuration optimization

The document describes how to optimize the configuration of synchronization tasks, to improve the data synchronization performance of DM.

## Full export

The configuration item related  to full export is `mydumpers`. The following describes how to configure the parameters  related to performance.

### `rows`

Setting the `rows` option enables single-table multi-thread concurrent export, the values is the maximum number of rows contained in each chunk exported. After opening, DM will first select a column as the split benchmark when the single table of MySQL is concurrently exported.  The selected priortiy is  primary key> unique index> normal index. Then selecting the target column, need to ensure that the column is an integer type  (such as `INT`, `MEDIUMINT`, `BIGINT`, etc.). 

The value of  `rows` can be set to 10000, and the specific value of setting can be changed according to the total number of rows in the table and the performance of the database. In  addition , need to set `threads` to control the number of  concurrent threads, the default value is 4, can be adjusted appropriately.

### `chunk-filesize`

DM full backup will split the data of each table into multiple chunks according to the value of the `chunk-filesize` parameter, and each chunk is saved in a file with a size of about `chunk-filesize`.  According to this parameter, the data is splitted into multiple files, so that the parallel processing logic of the DM Load processing unit can be used to increase the import speed. The default value of this parameter is 64 (the unit is MB). Under normal situation, it does not need to be set. Also can make appropriate adjustments according to the size of the entire size of data.

> **Note：**
>
> - The parameter value of `mydumpers` does not support updating after the synchronization task is created, so need to determine the value of each parameter before creating the task. If need to update, need to use dmctl stop the task to update the configuration file, and then re-create the task.

> - `mydumpers`.`threads` can be replaced with the configuration item `mydumper-thread` to simplify configuration.

> - If `rows` is set，DM will ingore the vaue of `chunk-filesize`. 

## Full import 

The configuration item related to full import is `loaders`. The following describes how to configure the parameters related to performance.

### `pool-size`

`pool-size`  is the setting of  the number of threads in the DM load phase. The default value is 16. Normally, there is no need to set it.  Also can make appropriate adjustments based on the size of full size of data and the performance of the database.

> **Note：**
>
> - The parameter value of `loaders` does not support updating after the synchronization task is created, so need to define the value of each parameter before creating the task. If need to update, so need to use dmctl stop the task to update the configuration file, and then re-create the task.

> - `loaders`.`pool-size` can be replaced with the configuration item `loader-thread` to simplify configuration.

## Incremental synchronization

The configuration related to incremental synchronization is `syncers`. The following describes how to configure the parameters related to performance.

### `worker-count`

`worker-count`  is the number of threads for concurrent synchronization DML in the DM sync phase. The default value is 16. If there is a high requirement for synchronization speed, the value the parameter can be adjusted appropriately.

### batch

`batch` is the number of DML included in each transaction when the data is synchronized to the  downstream database in the DM sync phase. The default value is 100. Normally, no adjustment is required.

> **Note：**
>
> - The parameter value of `syncers` does not support updating after the synchronization task is created, so need to define the value of each parameter before creating the task. If need to update, so need to use dmctl stop the task to update the configuration file, and then re-create the task.
> - `syncers`.`worker-count` can be replaced with the configuration item `syncer-thread` to simplify configuration.
> - The settings of `worker-count` and `batch` need to be adjusted according to the actual situation, for example: the network delay from the DM to the downstream database is high, can appropriately increase the `worker-count` and lower the `batch`.
