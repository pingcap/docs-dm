---
title: DM-worker Configuration File
summary: Learn the configuration file of DM-worker.
category: reference
aliases: ['/docs/tidb-data-migration/dev/dm-worker-configuration-file-full/']
---

# DM-worker Configuration File

This document introduces the configuration of DM worker, including a configuration file template and a description of each configuration parameter in this file.

## Configuration file template

The following is a configuration file template of the DM-worker:

```toml
# Worker Configuration.
name = "worker1"

# Log configuration.
log-level = "info"
log-file = "dm-worker.log"

# DM-worker listen address.
worker-addr = ":8262"
advertise-addr = "127.0.0.1:8262"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

## Configuration parameters

### Global

| Parameter        | Description                           |
| :------------ | :--------------------------------------- |
| `name` |  The name of the DM-worker. |
| `log-level` | Specifies a log level from `debug`, `info`, `warn`, `error`, and `fatal`. The default log level is `info`. |
| `log-file` | Specifies the log file directory. If this parameter is not specified, the logs are printed onto the standard output. |
| `worker-addr` | Specifies the address of DM-worker which provides services. You can omit the IP address and specify the port number only, such as ":8262". |
| `advertise-addr` | Specifies the address that DM-worker advertises to the outside world. |
| `join` | Corresponds to one or more [`master-addr`s](dm-master-configuration-file.md#global-configuration) in the DM-master configuration file. |
