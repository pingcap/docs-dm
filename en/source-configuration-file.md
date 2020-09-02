---
title: Upstream Database Configuration File
summary: Learn the configuration file of the upstream database
aliases: ['/docs/tidb-data-migration/dev/source-configuration-file/']
---

# Upstream Database Configuration File

This document introduces the configuration file of the upstream database, including a configuration file template and the description of each configuration parameter in this file.

## Configuration file template

The following is a configuration file template of the upstream database:

```yaml
source-id: "mysql-replica-01"

# Whether to enable GTID.
enable-gtid: false

from:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "ZqMLjZ2j5khNelDEfDoUhkD5aV5fIJOe0fiog9w=" # The user password of the upstream database. It is recommended to use the password encrypted with dmctl. 
  security:                       # The TLS configuration of the upstream database
    ssl-ca: "/path/to/ca.pem"
    ssl-cert: "/path/to/cert.pem"
    ssl-key: "/path/to/key.pem"
```

## Configuration parameters

This section describes each configuration parameter in the configuration file.

### Global configuration

| Parameter | Description |
| :------------ | :--------------------------------------- |
| `source-id` | Represents a MySQL instance ID. |
| `enable-gtid` | Determines whether to pull binlog from the upstream using GTID. The default value is `false`. In general, you do not need to configure `enable-gtid` manually. However, if GTID is enabled in the upstream database, and the primary/secondary switch is required, you need to set `enable-gtid` to `true`. |
| `host` | Specifies the host of the upstream database. |
| `port` | Specifies the port of the upstream database. |
| `user` | Specifies the username of the upstream database. |
| `password` | Specifies the user password of the upstream database. It is recommended to use the password encrypted with dmctl. |
| `security` | Specifies the TLS config of the upstream database. |
