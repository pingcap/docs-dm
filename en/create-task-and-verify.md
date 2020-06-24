---
title: Create and Verify a Data Replication Task
summary: Learn how to create a replication task to verify whether the cluster works well after the DM cluster is deployed.
category: how-to
---

# Create and Verify a Data Replication Task

This document describes how to create a simple data replication task to verify whether the cluster works well after the DM cluster is successfully deployed.

## Sample scenario

Suppose that you create a data replication task based on this sample scenario:

- Upstream MySQL instances (MySQL1 and MySQL2) are deployed on two servers, with binlog enabled in both instances.
- TiDB clusters are deployed on several servers, with the TiDB service exposed on one of the servers.
- A DM-master of the DM cluster provides service.

The descriptions of each node are as follows.

| Instance   | Server Address  | Port  |
| :---------- | :----------- | :--- |
| MySQL1     | 192.168.0.1 | 3306 |
| MySQL2     | 192.168.0.2 | 3306 |
| TiDB       | 192.168.0.3 | 4000 |
| DM-master  | 192.168.0.4 | 8261 |

Based on this scenario, the following sections describe how to create a data replication task.

## Configure the MySQL data source

Before starting a data replication task, you need to configure the MySQL data source.

### Encrypt the password

For safety reasons, it is recommended to configure and use encrypted passwords. You can use dmctl to encrypt the MySQL password. Suppose the password is "123456":

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

Record this encrypted value, which can be used for creating a MySQL data source in the following steps.

### Edit the source configuration file

Write the following configurations to `conf/source1.toml`.

```toml
# MySQL1 Configuration.
 
source-id = "mysql-replica-01"

# Indicates whether GTID is enabled
enable-gtid = false
 
[from]
host = "192.168.0.1"
user = "root"
password = "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
port = 3306
```

In MySQL2 data source, copy the above configurations to `conf/source2.toml`. You need to change `name` to `mysql-replica-02`, `host` to `192.168.0.2`, and change `password` and `port` to appropriate values.

### Create a source

To load the data source configurations of MySQL1 into the DM cluster using dmctl, run the following command in the terminal:

{{< copyable "shell-regular" >}}

```bash
./bin/dmctl --master-addr=192.168.0.4:8261 operate-source create conf/source1.toml
```

For MySQL2, replace the configuration file in the above command with that of MySQL2.

## Create a data replication task

Suppose that there are several sharded tables on both MySQL1 and MySQL2 instances. In these tables, their structures are identical; the prefix “t” is in their table names; the databases where they are located are named with the same prefix “sharding”; there is no conflict between the primary keys or the unique keys (in each sharded table, the primary keys or the unique keys are different from those of other tables). 

You need to replicate these sharded tables to the `db_target.t_target` table in TiDB.

1. Create the configuration file of the task:

    {{< copyable "" >}}

    ```yaml
    ---
    name: test
    task-mode: all
    is-sharding: true
    target-database:
    host: "192.168.0.3"
    port: 4000
    user: "root"
    password: "" # If the password is not empty, you need to configure the encrypted password using dmctl.

    mysql-instances:
    - source-id: "mysql-replica-01"
        black-white-list:  "instance"
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16
    - source-id: "mysql-replica-02"
        black-white-list:  "instance"
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16
    black-white-list:
    instance:
        do-dbs: ["~^sharding[\\d]+"]
        do-tables:
        -  db-name: "~^sharding[\\d]+"
        tbl-name: "~^t[\\d]+"
    routes:
    sharding-route-rules-table:
        schema-pattern: sharding*
        table-pattern: t*
        target-schema: db_target
        target-table: t_target
    sharding-route-rules-schema:
        schema-pattern: sharding*
        target-schema: db_target
    ```

2. To create a task using dmctl, write the above configurations to the `conf/task.yaml` file:

    {{< copyable "shell-regular" >}}

    ```bash
    ./bin/dmctl -master-addr 192.168.0.4:8261 start-task conf/task.yaml
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-01",
                "worker": "worker1"
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

Now, you have successfully created a task to replicate the sharded tables from the MySQL1 and MySQL2 instances to TiDB.

## Verify whether the cluster works well

You can modify data in the upstream MySQL sharded tables. Then use [sync-diff-inspector](https://docs.pingcap.com/tidb/v4.0/shard-diff) to check whether the upstream and downstream data are consistent. Consistent data means that the replication task works well (this also indicates that the cluster works well).
