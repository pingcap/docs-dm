---
title: Data Migration Configuration File Overview
summary: This document gives an overview of Data Migration configuration files.
category: reference
---

# Data Migration Configuration File Overview

This document gives an overview of configuration files of DM (Data Migration).

## DM process configuration files

- `dm-master.toml`: The configuration file of running the DM-master process, including the topology information and the logs of the DM-master and . For more details, refer to [DM-master Configuration File](dm-master-configuration-file.md).
- `dm-worker.toml`: The configuration file of running the DM-worker process, including the topology information and the logs of the DM-worker. For more details, refer to [DM-worker Configuration File](dm-worker-configuration-file.md).
- `source.toml`: The configuration of the upstream database such as MySQL and MariaDB. For more details, refer to [Upstream Database Configuration File](source-configuration-file.md).

## DM replication task configuration

### DM task configuration file

`task.yaml` is the configuration file for the data replication task . For more details, refer to [Task Configuration File](task-configuration-file.md).

### Data replication task creation

You can take the following steps to create a data replication task:

1. [Load the data source configuration into the DM cluster using dmctl](manage-replication-tasks.md#load-the-data-source-configuration).
2. Refer to the description in the [Task Configuration File](task-configuration-file.md) and create the configuration file `your_task.yaml`.
3. [Create the data replication task using dmctl](manage-replication-tasks.md#create-the-data-replication-task).

### Important concepts

This section shows description of some important concepts.

| Concept  | Description  | Configuration File  |
| :------ | :--------- | :------------- |
| `source-id`  | Uniquely identifies a MySQL or MariaDB instance, or a replication group with the master-slave structure. The maximum length of `source-id` is 32. | `source_id` of `source.toml`;<br> `source-id` of `task.yaml` |
| DM-master ID | Uniquely identifies a DM-master (by the `master-addr` parameter of `dm-master.toml`) | `master-addr` of `dm-master.toml` |
| DM-worker ID | Uniquely identifies a DM-worker (by the `worker-addr` parameter of `dm-worker.toml`) | `worker-addr` of `dm-worker.toml` |
