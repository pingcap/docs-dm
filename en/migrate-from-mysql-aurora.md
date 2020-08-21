---
title: Migrate from MySQL (Amazon Aurora)
summary: Learn how to migrate from MySQL (using a case of Amazon Aurora) to TiDB by using TiDB Data Migration (DM).
aliases: ['/docs/tidb-data-migration/dev/migrate-from-mysql-aurora/']
---

# Migrate from MySQL (Amazon Aurora)

This document describes how to migrate from [Amazon Aurora MySQL](https://aws.amazon.com/rds/aurora/details/mysql-details/?nc1=h_ls) to TiDB by using TiDB Data Migration (DM).

The Aurora cluster information in the example is as follows:

| Cluster | Endpoint | Port | Role | Version |
|:-------- |:--- | :--- | :--- |:---|
| Aurora-1 | test-dm-2-0.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | Writer | Aurora (MySQL)-5.7.12 |
| Aurora-1 | test-dm-2-0.cluster-ro-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | Reader | Aurora (MySQL)-5.7.12 |
| Aurora-2 | test-dm-2-0-2.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | Writer | Aurora (MySQL)-5.7.12 |
| Aurora-2 | test-dm-2-0-2.cluster-ro-czrtqco96yc6.us-east-2.rds.amazonaws.com | 3306 | Reader | Aurora (MySQL)-5.7.12 |

The data and migration plan of the Aurora cluster are as follows:

| Cluster | Database | Table | Migration |
|:---- |:---- | :--- | :--- |
| Aurora-1 | migrate_me | t1 | Yes |
| Aurora-1 | ignore_me | ignore_table | No |
| Aurora-2 | migrate_me | t2 | Yes |
| Aurora-2 | ignore_me | ignore_table | No |

The Aurora cluster users for migration are as follows:

| Cluster | User | Password |
|:---- |:---- | :--- |
| Aurora-1 | root | 12345678 |
| Aurora-2 | root | 12345678 |

The TiDB cluster information in the example is as follows. The cluster uses [TiDB Cloud](https://tidbcloud.com/) for one-click deployment:

| Node | Port | Version |
|:--- | :--- | :--- |
| tidb.6657c286.23110bc6.us-east-1.prod.aws.tidbcloud.com | 4000 | v4.0.2 |

The TiDB cluster users for migration are as follows:

| User | Password |
|:---- | :--- |
| root | 87654321 |

After migration, there are tables ``` `migrate_me`.`t1` ``` and ``` `migrate_me`.`t2` ``` in the TiDB cluster. The data of these tables is consistent with that of the Aurora cluster.

> **Note:**
>
> This migration does not involve the table combination function. To use the function, see [DM Shard Merge Scenario](scenarios.md#shard-merge-scenario).

## Step 1: Data migration precheck

To ensure a successful migration, pre-conditions need to be checked before starting the migration. Here lists the precheck and solutions to DM and Aurora components.

### DM nodes deployment 

As the core of data migration, DM needs to connect the upstream Aurora cluster and the downstream TiDB cluster. Therefore, use MySQL client and other methods to check whether the node where the DM is deployed can connect upstream and downstream. In addition, for DM requirements for software and hardware, see [DM Cluster Software and Hardware Recommendations](hardware-and-software-requirements.md).

### Aurora

DM relies on the `ROW` format of binlog during the incremental replication process. See [Enable binary for an Aurora Cluster](https://aws.amazon.com/premiumsupport/knowledge-center/enable-binary-logging-aurora/?nc1=h_ls)

To migrate data based on GTID, set both `gtid-mode` and `enforce_gtid_consistency` to `ON`. See [Configuring GTID-Based Replication for an Aurora MySQL Cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/mysql-replication-gtid.html#mysql-replication-gtid.configuring-aurora)

> **Note:**
>
> GTID-based data migration requires MySQL 5.7 (Aurora 2.04) version or later.

Besides the above Aurora-specific configuration, the upstream database meet other requirements for migrating MySQL. See [Checking Items](precheck.md#checking-items).

## Step 2: Deploy the DM cluster

DM can be deployed in a variety of ways. Currently, it is recommended to use TiUP to deploy a DM cluster. For the specific deployment method, see [Deploy DM cluster using TiUP](deploy-a-dm-cluster-using-tiup.md). The example has two data sources, so at least two DM-worker nodes need to be deployed. 

After deployment, you need to record the IP and service port of any DM-master node (`8261` by default) for `dmctl` to connect. This example uses `127.0.0.1:8261`. Check the DM status through TiUP using `dmctl`:

> **Note:**
>
> When using other methods to deploy DM, you can call `dmctl` in a similar way, see [Introduction to dmctl](dmctl-introduction.md).

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr 127.0.0.1:8261 list-member
```

The `master` and `worker` in the return value are consistent with the number of deployment:

```bash
{
    "result": true,
    "msg": "",
    "members": [
        {
            "leader": {
                ...
            }
        },
        {
            "master": {
                "msg": "",
                "masters": [
                    ...
                ]
            }
        },
        {
            "worker": {
                "msg": "",
                "workers": [
                    ...
                ]
            }
        }
    ]
}
```

## Step 3: Deploy the data source

> **Note:**
>
> The configuration file used by DM supports plain text or cipher text database passwords. It is recommended to use password encrypted with dmctl. For how to obtain the cipher text database password, see [Encrypt the database password using dmctl](manage-source.md#encrypt-the-database-password)
>>>>>>> Stashed changes

Save the following configuration files of data source according to the sample information, in which the value of `source-id` will be quoted when configuring the task in step 4. 

File `source1.yaml`:

```yaml
# Aurora-1
source-id: "aurora-replica-01"
enable-gtid: false
from:
  host: "test-dm-2-0.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com"
  user: "root"
  password: "12345678"
  port: 3306
```

File `source2.yaml`:

```yaml
# Aurora-2
source-id: "aurora-replica-02"
enable-gtid: false
from:
  host: "test-dm-2-0-2.cluster-czrtqco96yc6.us-east-2.rds.amazonaws.com"
  user: "root"
  password: "12345678"
  port: 3306
```

See [Replicate Data Using Data Migration](replicate-data-using-dm#step-3-create-data-source), and use `dmctl` to add two data sources through TiUP.

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr 127.0.0.1:8261 operate-source create dm-test/source1.yaml
tiup dmctl --master-addr 127.0.0.1:8261 operate-source create dm-test/source2.yaml
```

When the data source is successfully added, the return information of each data source includes a DM-worker bound to it.

```bash
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "aurora-replica-01",
            "worker": "one-dm-worker-ID"
        }
    ]
}
```

## Step 4: Configure the task

> **Note:**
>
> Since Aurora does not support FTWRL, writing needs to be suspended when only exporting data in full data migration mode. See [Instructions in AWS Official Website](https://aws.amazon.com/premiumsupport/knowledge-center/mysqldump-error-rds-mysql-mariadb/?nc1=h_ls). In both full data migration and incremental replication modes of the example, DM will automatically enable the `safe mode` to solve this problem. But there may still be data inconsistencies. To ensure data consistency, see [Instructions in AWS Official Website](https://aws.amazon.com/premiumsupport/knowledge-center/mysqldump-error-rds-mysql-mariadb/?nc1=h_ls), and set the last line `extra-args` of the following configuration file to empty.

This example migrates existing Aurora data and the newly added data is migrated to TiDB in real time. That is the **full data migration + incremental replication** mode. According to the above TiDB cluster information, the added `source-id`, and the table to be migrated, save the following task configuration file `task.yaml`:

```yaml
# The task name. You need to use a different name for each of the multiple tasks that run simultaneously.
name: "test"
# The full data migration plus incremental replication task mode.
task-mode: "all"
# The downstream TiDB configuration information.
target-database:
  host: "tidb.6657c286.23110bc6.us-east-1.prod.aws.tidbcloud.com"
  port: 4000
  user: "root"
  password: "87654321"

# Configuration of all the upstream MySQL instances required by the current data migration task.
mysql-instances:
- source-id: "aurora-replica-01"
  # The configuration item name of the block and allow lists of the schema or table to be migrated, used to quote the global block and allow lists configuration. For global configuration, see the `block-allow-list` below.
  block-allow-list: "global"
  mydumper-config-name: "global"

- source-id: "aurora-replica-02"
  block-allow-list: "global"
  mydumper-config-name: "global"

# The configuration of block and allow lists.
block-allow-list:
  global:                             # Quoted by block-allow-list: "global" above
    do-dbs: ["migrate_me"]            # The allow list of the upstream table to be migrated. Database tables outside the allow list will not be migrated.

# Dumper configuration.
mydumpers:
   global:                             # Quoted by mydumper-config-name: "global" above
    extra-args: "--consistency none"  # Aurora does not support FTWRL, you need to configure this option to bypass.
```

## Step 5: Start the task

Start tasks using `dmctl` through TiUP.

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr 127.0.0.1:8261 start-task task.yaml --remove-meta
```

The return information when the task is successfully started:

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "aurora-replica-01",
            "worker": "one-dm-worker-ID"
        },
        {       
            "result": true,
            "msg": "",
            "source": "aurora-replica-02",
            "worker": "another-dm-worker-ID"
        }
    ]
}
```

## Step 6: Query the task and verify the data

Use `dmctl` through TiUP to query information about the on-going migration task and task status.

{{< copyable "shell-regular" >}}


```bash
tiup dmctl --master-addr 127.0.0.1:8261 query-status
```

The return information of when the task normally operate:

```
{
    "result": true,
    "msg": "",
    "tasks": [
        {
            "taskName": "test",
            "taskStatus": "Running",
            "sources": [
                "aurora-replica-01",
                "aurora-replica-02"
            ]
        }
    ]
}
```

Users can query data in the downstream, modify data in Aurora, and verify data replication in TiDB.
