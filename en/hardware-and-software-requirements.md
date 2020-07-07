---
title: Software and Hardware Requirements
summary: Learn the software and hardware requirements for DM cluster.
category: how-to
aliases: ['/docs/tidb-data-migration/dev/hardware-and-software-requirements/']
---

# Software and Hardware Requirements

TiDB Data Migration (DM) supports the Linux operating system, and can be well deployed and run on the Intel architecture server and mainstream virtualization environment.

## Linux OS version requirements

| Linux OS Platform       | Version         |
| :----------------------- | :----------:   |
| Red Hat Enterprise Linux | 7.3 or later   |
| CentOS                   | 7.3 or later   |
| Oracle Enterprise Linux  | 7.3 or later   |
| Ubuntu LTS               | 16.04 or later |

> **Note:**
>
> The Linux OS platforms above can run in physical servers as well as mainstream virtualization environments like VMware, KVM, and XEN.

## Recommended server requirements

DM can be deployed and run on a 64-bit generic hardware server platform (Intel x86-64 architecture). For servers used in the development, testing, and production environments, this section illustrates recommended hardware configurations (these do not include the resources used by the operating system).

### Development and test environments

| Component | CPU | Memory | Local Storage | Network | Number of Instances (Minimum Requirement) |
| --- | --- | --- | --- | --- | --- |
| DM-master | 4 core+ | 8 GB+ | SAS, 200 GB+ | Gigabit network card | 1 |
| DM-worker | 8 core+ | 16 GB+ | SAS, 200 GB+ (Greater than the size of the migrated data) | Gigabit network card | The number of upstream MySQL instances |

> **Note:**
>
> - In the test environment, DM-master and DM-worker used for functional verification can be deployed on the same server.
> - To prevent interference with the accuracy of the performance test results, it is **not recommended** to use low-performance storage and network hardware configurations.
> - If you need to verify the function only, you can deploy a DM-master on a single machine. The number of DM-worker deployed depends on the number of upstream MySQL instances to be replicated.
> - DM-worker stores full data in the `dump` and `load` phases. Therefore, the disk space for DM-worker needs to be greater than the total amount of data to be migrated. If the relay log is enabled for the replication task, DM-worker needs additional disk space to store upstream binlog data.

### Production environment

| Component | CPU | Memory | Hard Disk Type | Network | Number of Instances (Minimum Requirement) |
| --- | --- | --- | --- | --- | --- |
| DM-master | 4 core+ | 8 GB+ | SAS, 200 GB+ | Gigabit network card | 3 |
| DM-worker | 16 core+ | 32 GB+ | SSD, 200 GB+ (Greater than the size of the migrated data) | 10 Gigabit network card | Greater than the number of upstream MySQL instances |
| Monitor | 8 core+ | 16 GB+ | SAS, 200 GB+ | Gigabit network card | 1 |

> **Note:**
>
> - In the production environment, DM-master and DM-worker can be deployed and run on the same server.
> - It is recommended to use server configurations higher than those mentioned in the table above in production environments.
