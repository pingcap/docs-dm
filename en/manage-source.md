---
title: Manage Upstream MySQL Instances
summary: Learn how to manage upstream MySQL instances in TiDB Data Migration.
---

# Manage Upstream MySQL Instances

This document introduces how to manage upstream MySQL instances, including encrypting the MySQL password and managing data source configurations using [dmctl](dmctl-introduction.md).

## Encrypt the MySQL password

In DM configuration files, it is recommended to use the password encrypted with dmctl. For one original password, the encrypted password is different after each encryption.

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 123456
```

```
VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=
```

## Load the data source configurations

You can use the `operate-source` command to load the data source configurations to the DM cluster.

{{< copyable "" >}}

```bash
help operate-source
```

```
create/update/stop/show upstream MySQL/MariaDB source

Usage:
  dmctl operate-source <operate-type> [config-file ...] [--print-sample-config] [flags]

Flags:
  -h, --help                  help for operate-source
  -p, --print-sample-config   print sample config file of source

Global Flags:
  -s, --source strings   MySQL Source ID
```

## Usage example

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

For the configuration of `source.yaml`, refer to [Upstream Database Configuration File Introduction](source-configuration-file.md).

## Flags description

+ `create`: Creates an upstream database source.

+ `update`: Updates an upstream database source.

+ `stop`: Stops an upstream database source.

+ `show`: Shows the added data source and the corresponding DM-worker.

+ `config-file`: Specifies the file path of `source.yaml` and can pass multiple file paths.

+ `--print-sample-config`: Prints the sample configuration file. This parameter ignores other parameters.

## Returned results

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
