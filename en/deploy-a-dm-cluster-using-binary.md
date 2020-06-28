---
title: Deploy Data Migration Using DM Binary
summary: Learn how to deploy a Data Migration cluster using DM binary.
category: how-to
aliases: ['/docs/tidb-data-migration/dev/deploy-a-dm-cluster-using-binary/']
---

# Deploy Data Migration Using DM Binary

This document introduces how to quickly deploy the Data Migration (DM) cluster using DM binary.

## Preparations

Download the official binary from [here](https://pingcap.com/docs/dev/reference/tools/download/#tidb-dm-data-migration).

The downloaded files have two subdirectories, `bin` and `conf`. The `bin` directory contains the binary files of DM-master, DM-worker, and dmctl. The `conf` directory contains the sample configuration files.

## Sample scenario

Suppose that you are going to deploy a DM cluster based on this sample scenario:

+ Two MySQL instances are deployed on two servers.
+ One TiDB instance is deployed on one server (in the mocktikv mode).
+ Two DM-worker nodes and three DM-master nodes are deployed on five servers.

Here is the address of each node:

| Instance or node        | Server address   |
| :---------- | :----------- |
| MySQL1     | 192.168.0.1 |
| MySQL2     | 192.168.0.2 |
| TiDB       | 192.168.0.3 |
| DM-master1 | 192.168.0.4 |
| DM-master2 | 192.168.0.5 |
| DM-master3 | 192.168.0.6 |
| DM-worker1 | 192.168.0.7 |
| DM-worker2 | 192.168.0.8 |

You need to enable the binlog on MySQL1 and on MySQL2.

Based on this scenario, the following sections describe how to deploy the DM cluster.

### Deploy DM-master

You can configure DM-master by using [command-line parameters](#dm-master-command-line-parameters) or [the configuration file](#dm-master-configuration-file).

#### DM-master command-line parameters

The following is the description of DM-master command-line parameters:

```bash
./bin/dm-master --help
```

```
Usage of dm-master:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${master-addr}")
  -advertise-peer-urls string
        advertise URLs for peer traffic (default "${peer-urls}")
  -config string
        path to config file
  -data-dir string
        path to the data directory (default "default.${name}")
  -initial-cluster string
        initial cluster configuration for bootstrapping, e.g. dm-master=http://127.0.0.1:8291
  -join string
        join to an existing cluster (usage: cluster's "${master-addr}" list, e.g. "127.0.0.1:8261,127.0.0.1:18261"
  -log-file string
        log file path
  -master-addr string
        master API server and status addr
  -name string
        human-readable name for this DM-master member
  -peer-urls string
        URLs for peer traffic (default "http://127.0.0.1:8291")
  -print-sample-config
        print sample config file of dm-worker
```

> **Note:**
>
> In some situations, you cannot use the above method to configure DM-master because some configurations are not exposed to the command line. In such cases, use the configuration file instead.

#### DM-master configuration file

The following is the configuration file of DM-master. It is recommended that you configure DM-master by using this method.

1. Write the following configuration to `conf/dm-master1.toml`:

      ```toml
      # Master Configuration.
      name = "master1"

      # Log configurations.
      log-level = "info"
      log-file = "dm-master.log"

      # The listening address of DM-master.
      master-addr = ":8261"

      # The peer URLs of DM-master.
      peer-urls = "192.168.0.4:8291"

      # The value of `initial-cluster` is the combination of the `advertise-peer-urls` value of all DM-master nodes in the initial cluster.
      initial-cluster = "master1=http://192.168.0.4:8291,master2=http://192.168.0.5:8291,master3=http://192.168.0.6:8291"
      ```

2. Execute the following command in the terminal to run DM-master:

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dm-master -config conf/dm-master1.toml
      ```

3. For DM-master2 and DM-master3, change `name` in the configuration file to `master2` and `master3` respectively, and change `peer-urls` to `192.168.0.5:8291` and `192.168.0.6:8291` respectively. Then repeat Step 2.

### Deploy DM-worker

You can configure DM-worker by using [command-line parameters](#dm-worker-command-line-parameters) or [the configuration file](#dm-worker-configuration-file).

#### DM-worker command-line parameters

The following is the description of the DM-worker command-line parameters:

{{< copyable "shell-regular" >}}

```bash
./bin/dm-worker --help
```

```
Usage of worker:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${worker-addr}")
  -config string
        path to config file
  -join string
        join to an existing cluster (usage: dm-master cluster's "${master-addr}")
  -keepalive-ttl int
        dm-worker's TTL for keepalive with etcd (in seconds) (default 10)
  -log-file string
        log file path
  -name string
        human-readable name for DM-worker member
  -print-sample-config
        print sample config file of dm-worker
  -worker-addr string
        listen address for client traffic
```

> **Note:**
>
> In some situations, you cannot use the above method to configure DM-worker because some configurations are not exposed to the command line. In such cases, use the configuration file instead.

#### DM-worker configuration file

The following is the DM-worker configuration file. It is recommended that you configure DM-worker by using this method.

1. Write the following configuration to `conf/dm-worker1.toml`:

      ```toml
      # Worker Configuration.
      name = "worker1"

      # Log configuration.
      log-level = "info"
      log-file = "dm-worker.log"

      # DM-worker address.
      worker-addr = ":8262"

      # The master-addr configuration of the DM-master nodes in the cluster.
      join = "192.168.0.4:8261,192.168.0.5:8261,192.168.0.6:8261"
      ```

2. Execute the following command in the terminal to run DM-worker:

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dm-worker -config conf/dm-worker1.toml
      ```

3. For DM-worker2, change `name` in the configuration file to `worker2`. Then repeat Step 2.

### Configure MySQL source

Before creating a data replication task, configure the MySQL source first. For safety reasons, you must configure and use the encrypted password.

1. Encrypt the MySQL password using dmctl. Suppose the password is "123456":

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dmctl --encrypt "123456"
      ```

      Then, you get the encrypted password as shown below:

      ```
      fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
      ```

2. Record this encrypted password, which is used for configuring MySQL1. The following is the configuration file of MySQL1. You need to write the following configuration to `conf/source1.toml`.

      ```toml
      # MySQL1 Configuration.
      source-id = "mysql-replica-01"

      # Determines whether to enable GTID.
      enable-gtid = false

      [from]
      host = "192.168.0.1"
      user = "root"
      password = "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
      port = 3306
      ```

3. Run the following command in the terminal to load the data source configuration into the DM cluster using dmctl:

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dmctl --master-addr=192.168.0.4:8261 operate-source create conf/source1.toml
      ```

4. For MySQL2, change `name` in the configuration file to `mysql-replica-02`, `host` to `192.168.0.2`, and change `password` and `port` to the corresponding value. Then repeat Step 3.

Now, a DM cluster is successfully deployed.

### Create a data replication task

Suppose that there are several sharded tables on both MySQL1 and MySQL2 instances. These tables have the same structure and the same prefix "t" in their table names. The databases where they are located are named with the same prefix "sharding". In each sharded table, the primary key and unique key are different from those of all other tables.

Now you need to replicate these sharded tables to the `db_target.t_target` table in TiDB.

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
      password: "" # if the password is not empty, you also need to configure the encrypted password using dmctl

    mysql-instances:
      - source-id: "mysql-replica-01"
        block-allow-list:  "instance"  # Use black-white-list if the DM's version <= v2.0.0-beta.2.
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16

      - source-id: "mysql-replica-02"
        block-allow-list:  "instance"  # Use black-white-list if the DM's version <= v2.0.0-beta.2.
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16

    block-allow-list:  # Use black-white-list if the DM's version <= v2.0.0-beta.2.
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

2. Write the above configuration to the `conf/task.yaml` file and create the task using dmctl:

    {{< copyable "shell-regular" >}}

    ```bash
    ./bin/dmctl -master-addr 192.168.0.4:8261
    ```

    ```
    Welcome to dmctl
    Release Version: v1.0.0-69-g5134ad1
    Git Commit Hash: 5134ad19fbf6c57da0c7af548f5ca2a890bddbe4
    Git Branch: master
    UTC Build Time: 2019-04-29 09:36:42
    Go Version: go version go1.12 linux/amd64
    »
    ```

    {{< copyable "" >}}

    ```bash
    » start-task conf/task.yaml
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
