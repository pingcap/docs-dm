---
title: Data Migration Configuration File Overview
summary: This document gives an overview of Data Migration configuration files.
aliases: ['/docs/tidb-data-migration/stable/config-overview/','/docs/tidb-data-migration/v1.0/config-overview/','/docs/dev/reference/tools/data-migration/configure/overview/','/docs/v3.1/reference/tools/data-migration/configure/overview/','/docs/v3.0/reference/tools/data-migration/configure/overview/','/docs/v2.1/reference/tools/data-migration/configure/overview/']
---

# Data Migration Configuration File Overview

This document gives an overview of configuration files of DM (Data Migration).

## DM process configuration files

- `inventory.ini`: The configuration file of deploying DM using DM-Ansible. You need to edit it based on your machine topology. For details, see [Edit the `inventory.ini` file to orchestrate the DM cluster](deploy-a-dm-cluster-using-ansible.md#step-7-edit-the-inventoryini-file-to-orchestrate-the-dm-cluster).
- `dm-master.toml`: The configuration file of running the DM-master process, including the topology information of the DM cluster and the corresponding relationship between the MySQL instance and DM-worker (must be one-to-one relationship). When you use DM-Ansible to deploy DM, `dm-master.toml` is generated automatically. Refer to [DM-master Configuration File](dm-master-configuration-file.md) to see more details.
- `dm-worker.toml`: The configuration file of running the DM-worker process, including the upstream MySQL instance configuration and the relay log configuration. When you use DM-Ansible to deploy DM, `dm-worker.toml` is generated automatically. Refer to [DM-worker Configuration File](dm-master-configuration-file.md) to see more details.

## DM migration task configuration

### DM task configuration file

<<<<<<< HEAD
When you use DM-Ansible to deploy DM, you can find the following task configuration file template in `<path-to-dm-ansible>/conf`:

- `task.yaml.exmaple`: The standard configuration file of the data replication task (a specific task corresponds to a `task.yaml`). For the introduction of the configuration file, see [Task Configuration File](task-configuration-file.md).
=======
`task.yaml` is the configuration file for the data migration task. For more details, refer to [Task Configuration File](task-configuration-file.md).
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

### Data migration task creation

<<<<<<< HEAD
You can perform the following steps to create a data replication task based on `task.yaml.example`:

1. Copy `task.yaml.example` as `your_task.yaml`.
2. Refer to the description in the [Task Configuration File](task-configuration-file.md) and modify the configuration in `your_task.yaml`.
3. Create your data replication task using dmctl.
=======
You can take the following steps to create a data migration task:

1. [Load the data source configuration into the DM cluster using dmctl](manage-source.md#load-the-data-source-configurations).
2. Refer to the description in the [Task Configuration File](task-configuration-file.md) and create the configuration file `your_task.yaml`.
3. [Create the data migration task using dmctl](create-task.md).
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)

### Important concepts

This section shows description of some important concepts.

| Concept  | Description  | Configuration File  |
| :------ | :--------- | :------------- |
<<<<<<< HEAD
| `source-id`  | Uniquely identifies a MySQL or MariaDB instance, or a replication group with the primary-secondary structure. The maximum length of `source-id` is 32. | `source_id` of `inventory.ini`;<br/> `source-id` of `dm-master.toml`;<br/> `source-id` of `task.yaml` |
| DM-worker ID | Uniquely identifies a DM-worker (by the `worker-addr` parameter of `dm-worker.toml`) | `worker-addr` of `dm-worker.toml`;<br/> the `-worker`/`-w` flag of the dmctl command line |
=======
| `source-id`  | Uniquely represents a MySQL or MariaDB instance, or a migration group with the primary-secondary structure. The maximum length of `source-id` is 32. | `source_id` of `source.yaml`;<br/> `source-id` of `task.yaml` |
| DM-master ID | Uniquely represents a DM-master (by the `master-addr` parameter of `dm-master.toml`) | `master-addr` of `dm-master.toml` |
| DM-worker ID | Uniquely represents a DM-worker (by the `worker-addr` parameter of `dm-worker.toml`) | `worker-addr` of `dm-worker.toml` |
>>>>>>> e32acdc... en: Update descriptions about 迁移 & 同步 to make it clearer (#312)
