---
title: Upstream Database Configuration File
summary: Learn the configuration file of the upstream database
category: reference
---

# Upstream Database Configuration File

This document introduces the configuration files of the upstream database, including a configuration file template and instructions about what the configuration parameters mean.

## Configuration file template

The following is a configuration file template of the upstream database:

```toml
source-id = "mysql-replica-01"

# Enable GTID

enable-gtid = false

[from]
host = "127.0.0.1"
port = 3306
user = "root"
password = "ZqMLjZ2j5khNelDEfDoUhkD5aV5fIJOe0fiog9w=" # The password here refers to the dmctl-encrypted password of the upstream MySQL user
```

## Configuration parameters

### Global configuration

| Parameter | Description |
| :------------ | :--------------------------------------- |
| `source-id` | Identifies a MySQL instance. |
| `enable-gtid` | Determines whether to pull binlog from the upstream using GTID. The default value is `false`. In general, you don't need to configure `enable-gtid` manually. However, if GTID is enabled in the upstream database, and the master/slave switch is required, you need to set `enable-gtid` to `true`. |
| `host` | Specifies the host of the upstream database. |
| `port` | Specifies the port of the upstream database. |
| `user` | Specifies the username of the upstream database. |
| `password` | Specifies the user password of the upstream database. Note that the password here refers to the dmctl-encrypted password. |
