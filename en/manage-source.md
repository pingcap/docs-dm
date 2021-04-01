---
title: Manage Upstream MySQL Instances
summary: Learn how to manage upstream MySQL instances in TiDB Data Migration.
---

# Manage Upstream MySQL Instances

This document introduces how to manage upstream MySQL instances, including encrypting the MySQL password, managing data source configurations, and change binding relations of upstream MySQL instances and DM-workers using [dmctl](dmctl-introduction.md).

## Encrypt the database password

In DM configuration files, it is recommended to use the password encrypted with dmctl. For one original password, the encrypted password is different after each encryption.

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 'abc!@#123'
```

```
MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
```

## Load, list, remove the data source

You can use the `operate-source` command to load, list or remove the data source configurations to the DM cluster.

{{< copyable "" >}}

```bash
help operate-source
```

```
`create`/`update`/`stop`/`show` upstream MySQL/MariaDB source.

Usage:
  dmctl operate-source <operate-type> [config-file ...] [--print-sample-config] [flags]

Flags:
  -h, --help                  help for operate-source
  -p, --print-sample-config   print sample config file of source

Global Flags:
  -s, --source strings   MySQL Source ID
```

### Usage example

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

For the configuration of `source.yaml`, refer to [Upstream Database Configuration File Introduction](source-configuration-file.md).

### Flags description

+ `create`: Creates one or more upstream database source(s). When creating multiple data sources fails, DM rolls back to the state where the command was not executed.

+ `update`: Updates an upstream database source.

+ `stop`: Stops one or more upstream database source(s). When stopping multiple data sources fails, some data sources might be stopped.

+ `show`: Shows the added data source and the corresponding DM-worker.

+ `config-file`: Specifies the file path of `source.yaml` and can pass multiple file paths.

+ `--print-sample-config`: Prints the sample configuration file. This parameter ignores other parameters.

### Returned results

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
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
            "worker": "dm-worker-1"
        }
    ]
}
```

## Check effective data source configurations in DM-master

> **Note:**
>
> The `get-config` command is only supported in DM v2.0.1 and later versions.

If you know the `source-id`, you could use `dmctl --master-addr <master-addr> get-config source <source-id>` to get the data source configuration.

{{< copyable "" >}}

```bash
get-config source mysql-replica-01
```

```
{
    "result": true,
    "msg": "",
    "cfg": "enable-gtid: false\nauto-fix-gtid: false\nrelay-dir: \"\"\nmeta-dir: \"\"\nflavor: mysql\ncharset: \"\"\nenable-relay: false\nrelay-binlog-name: \"\"\nrelay-binlog-gtid: \"\"\nsource-id: mysql-replica-01\nfrom:\n  host: 172.16.4.222\n  port: 3306\n  user: dm_user\n  password: '******'\n  max-allowed-packet: null\n  session: {}\n  security: null\npurge:\n  interval: 3600\n  expires: 0\n  remain-space: 15\nchecker:\n  check-enable: true\n  backoff-rollback: 5m0s\n  backoff-max: 5m0s\n  check-interval: 5s\n  backoff-min: 1s\n  backoff-jitter: true\n  backoff-factor: 2\nserver-id: 429595182\ntracer: {}\n"
}
```

If you don't know the `source-id`, you could use `dmctl --master-addr <master-addr> operate-source show` to list all data sources.

{{< copyable "" >}}

```bash
operate-source show
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "source is added but there is no free worker to bound",
            "source": "mysql-replica-02",
            "worker": ""
        },
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

## Change the binding relations of upstream MySQL instances and DM-workers

You can use the `transfer-source` command change the binding relations of upstream MySQL instances and DM-workers.

{{< copyable "" >}}

```bash
help transfer-source
```

```
Transfers a upstream MySQL/MariaDB source to a free worker.
Usage:
  dmctl transfer-source <source-id> <worker-id> [flags]
Flags:
  -h, --help   help for transfer-source
Global Flags:
  -s, --source strings   MySQL Source ID.
```

Before changing, DM will check if the unbinding worker is still has running tasks. If so, please [pause tasks](pause-task.md) before the changing, and [resume tasks](resume-task.md) after the changing.

### Usage example


If you don't know the binding relation of DM-workers, you could use `dmctl --master-addr <master-addr> list-member --worker` to list current binding relations.

{{< copyable "" >}}

```bash
list-member --worker
```

```
{
    "result": true,
    "msg": "",
    "members": [
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "dm-worker-1",
                        "addr": "127.0.0.1:8262",
                        "stage": "bound",
                        "source": "mysql-replica-01"
                    },
                    {
                        "name": "dm-worker-2",
                        "addr": "127.0.0.1:8263",
                        "stage": "free",
                        "source": ""
                    }
                ]
            }
        }
    ]
}
```

In this example `mysql-replica-01` is bound to `dm-worker-1`. The below command should transfer the source to bind to `dm-worker-2`.

{{< copyable "" >}}

```bash
transfer-source mysql-replica-01 dm-worker-2
```

```
{
    "result": true,
    "msg": ""
}
```

Check the command has taken effect by `dmctl --master-addr <master-addr> list-member --worker`.

{{< copyable "" >}}

```bash
list-member --worker
```

```
{
    "result": true,
    "msg": "",
    "members": [
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "dm-worker-1",
                        "addr": "127.0.0.1:8262",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "dm-worker-2",
                        "addr": "127.0.0.1:8263",
                        "stage": "bound",
                        "source": "mysql-replica-01"
                    }
                ]
            }
        }
    ]
}
```
