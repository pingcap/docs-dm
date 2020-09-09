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

When you use DM-Ansible to deploy DM, you can find the following task configuration file template in `<path-to-dm-ansible>/conf`:

- `task.yaml.exmaple`: The standard configuration file of the data migration task (a specific task corresponds to a `task.yaml`). For the introduction of the configuration file, see [Task Configuration File](task-configuration-file.md).

### Data migration task creation

You can perform the following steps to create a data migration task based on `task.yaml.example`:

1. Copy `task.yaml.example` as `your_task.yaml`.
2. Refer to the description in the [Task Configuration File](task-configuration-file.md) and modify the configuration in `your_task.yaml`.
3. Create your data migration task using dmctl.

### Important concepts

This section shows description of some important concepts.

| Concept  | Description  | Configuration File  |
| :------ | :--------- | :------------- |
| `source-id`  | Uniquely identifies a MySQL or MariaDB instance, or a replication group with the primary-secondary structure. The maximum length of `source-id` is 32. | `source_id` of `inventory.ini`;<br/> `source-id` of `dm-master.toml`;<br/> `source-id` of `task.yaml` |
| DM-worker ID | Uniquely identifies a DM-worker (by the `worker-addr` parameter of `dm-worker.toml`) | `worker-addr` of `dm-worker.toml`;<br/> the `-worker`/`-w` flag of the dmctl command line |
