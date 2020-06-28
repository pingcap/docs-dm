---
title: Quick Start Guide for TiDB Data Migration
summary: Learn how to quickly deploy a DM cluster using binary packages.
category: how-to
aliases: ['/docs/tidb-data-migration/dev/get-started/']
---

# Quick Start Guide for TiDB Data Migration

This document describes how to quickly deploy a [TiDB Data Migration](https://github.com/pingcap/dm) (DM) cluster using binary packages.

## Deploy instances locally

Deploy MySQL, TiDB, DM-master, and DM-worker instances locally. The detailed information of each instance is as follows:

| Instance        | Server Address   | Port |
| :---------- | :----------- | :----------- |
| MySQL1     | 127.0.0.1 | 3306 |
| MySQL2     | 127.0.0.1 | 3307 |
| TiDB       | 127.0.0.1 | 4000, 10080 |
| DM-master1 | 127.0.0.1 | 8261, 8291 |
| DM-master2 | 127.0.0.1 | 8361, 8292 |
| DM-master3 | 127.0.0.1 | 8461, 8293 |
| DM-worker1 | 127.0.0.1 | 8262 |
| DM-worker2 | 127.0.0.1 | 8263 |
| DM-worker3 | 127.0.0.1 | 8264 |

## Download binary

Before deploying DM, you need to download the binary package, run the upstream and downstream databases, and prepare the data.

### Download DM binary package

Download the latest (DM v2.0) version of the binary package or compile the package manually.

#### Method 1: Download the latest version of binary package

{{< copyable "shell-regular" >}}

```bash
wget http://download.pingcap.org/dm-nightly-linux-amd64.tar.gz
tar -xzvf dm-nightly-linux-amd64.tar.gz
cd dm-nightly-linux-amd64
```

#### Method 2: Compile the latest version of binary package

{{< copyable "shell-regular" >}}

```bash
git clone https://github.com/pingcap/dm.git
cd dm
make
```

**Optional**: Add the downloaded/compiled binary to the environment variable `PATH` for easier deployment.

{{< copyable "shell-regular" >}}

```bash
DM_PATH=`pwd` && export PATH=$PATH:$DM_PATH/bin
```

### Run upstream MySQL

Run two MySQL client services. To start MySQL services using Docker, execute the following commands:

{{< copyable "shell-regular" >}}

```bash
docker run --rm --name mysql-3306 -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3306 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3306.log 2>&1 &
docker run --rm --name mysql-3307 -p 3307:3307 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3307 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3307.log 2>&1 &
```

### Prepare data

- Write [sample data](https://github.com/pingcap/dm/blob/52205177910024c8b66c7a6ef05b1f9501c5901b/tests/ha/data/db1.prepare.sql) to `mysql-3306`.

- Write [sample data](https://github.com/pingcap/dm/blob/52205177910024c8b66c7a6ef05b1f9501c5901b/tests/ha/data/db2.prepare.sql) to `mysql-3307`.

### Run downstream TiDB

To run a TiDB server in the mocktikv mode, execute the following commands:

{{< copyable "shell-regular" >}}

```bash
wget https://download.pingcap.org/tidb-v4.0.0-rc.2-linux-amd64.tar.gz
tar -xzvf tidb-v4.0.0-rc.2-linux-amd64.tar.gz
mv tidb-v4.0.0-rc.2-linux-amd64/bin/tidb-server ./
./tidb-server -P 4000 --store mocktikv --log-file "./tidb.log" &
```

## Deploy DM-master

1. Create three directories: master1, master2, and master3.
2. Create a DM-master configuration file in each directory. The configuration files are as follows:

    `master1/dm-master1.toml`:

    ```toml
    # DM-master1 Configuration.
    name = "master1"
    master-addr = ":8261"
    advertise-addr = "127.0.0.1:8261"
    peer-urls = "127.0.0.1:8291"
    initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
    ```

    `master2/dm-master2.toml`:

    ```toml
    # DM-master2 Configuration.
    name = "master2"
    master-addr = ":8361"
    advertise-addr = "127.0.0.1:8361"
    peer-urls = "127.0.0.1:8292"
    initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
    ```

    `master3/dm-master3.toml`:

    ```toml
    # DM-master3 Configuration.
    name = "master3"
    master-addr = ":8461"
    advertise-addr = "127.0.0.1:8461"
    peer-urls = "127.0.0.1:8293"
    initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
    ```

3. Enter the directories of master1, master2, and master3 respectively, and execute the following commands to start each DM-master:

    {{< copyable "shell-regular" >}}

    ```bash
    nohup dm-master --config=dm-master1.toml --log-file=dm-master1.log >> dm-master1.log 2>&1 &
    nohup dm-master --config=dm-master2.toml --log-file=dm-master2.log >> dm-master2.log 2>&1 &
    nohup dm-master --config=dm-master3.toml --log-file=dm-master3.log >> dm-master3.log 2>&1 &
    ```

## Deploy DM-worker

1. Create three directories: worker1, worker2, and worker3.
2. Create a DM-worker configuration file in each directory. The configuration files are as follows:

    `worker1/dm-worker1.toml`:

    ```toml
    # DM-worker1 Configuration
    name = "worker1"
    worker-addr="0.0.0.0:8262"
    advertise-addr="127.0.0.1:8262"
    join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
    ```

    `worker2/dm-worker2.toml`:

    ```toml
    # DM-worker2 Configuration
    name = "worker2"
    worker-addr="0.0.0.0:8263"
    advertise-addr="127.0.0.1:8263"
    join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
    ```

    `worker3/dm-worker3.toml`:

    ```toml
    # DM-worker3 Configuration
    name = "worke3"
    worker-addr="0.0.0.0:8264"
    advertise-addr="127.0.0.1:8264"
    join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
    ```

3. Enter the directories of worker1, worker2, and worker3 respectively, and execute the following commands to start each DM-worker:

    {{< copyable "shell-regular" >}}

    ```bash
    nohup dm-worker --config=dm-worker1.toml --log-file=dm-worker1.log >> dm-worker1.log 2>&1 &
    nohup dm-worker --config=dm-worker2.toml --log-file=dm-worker2.log >> dm-worker2.log 2>&1 &
    nohup dm-worker --config=dm-worker3.toml --log-file=dm-worker3.log >> dm-worker3.log 2>&1 &
    ```

## Check deployment status

To check whether the DM cluster has been deployed successfully, execute the following command:

{{< copyable "shell-regular" >}}

```bash
dmctl --master-addr=127.0.0.1:8261 list-member
```

You need to check the following information in the returned result:

- Whether there is a `leader` item;
- Whether the `master` and the `worker` items include all topology information of DM-master and DM-worker.

A normal DM cluster returns the following information:

```bash
{
    "result": true,
    "msg": "",
    "members": [
        {
            "leader": {
                "msg": "",
                "name": "master1",
                "addr": "127.0.0.1:8261"
            }
        },
        {
            "master": {
                "msg": "",
                "masters": [
                    {
                        "name": "master1",
                        "memberID": "11007177379717700053",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8291"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8261"
                        ]
                    },
                    {
                        "name": "master2",
                        "memberID": "12007177379717800042",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8292"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8361"
                        ]
                    },
                    {
                        "name": "master3",
                        "memberID": "13007157379717700087",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8293"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8461"
                        ]
                    },
                ]
            }
        },
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "worker1",
                        "addr": "127.0.0.1:8262",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "worker2",
                        "addr": "127.0.0.1:8263",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "worker3",
                        "addr": "127.0.0.1:8264",
                        "stage": "free",
                        "source": ""
                    }
                ]
            }
        }
    ]
}
```
